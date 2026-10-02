# M1 操作手冊：在目標 GPU 上重現 LeWorldModel

在單張 8 GB 顯示記憶體的 GPU 上執行的步驟，對應 [roadmap.md](roadmap.md) 的 M1 與 [evaluation.md](../design/evaluation.md) 的檢驗 0.1。結果用來決定 [decisions/0003](../decisions/0003-lewm-as-world-model-starting-point.md) 是否接受。

以下指令依 LeWorldModel 官方程式碼 2026-05-22 的提交 8edfeb3 與 stable-worldmodel 0.1.1 確認。若你取得的版本行為不同，以官方 README 為準，並把差異記在結果中。

## 流程總覽

| 步驟 | 內容 | 需要先鎖定門檻嗎 |
| --- | --- | --- |
| 1 | 記錄硬體與軟體版本 | 否 |
| 2 | 安裝 LeWorldModel | 否 |
| 3 | 下載 TwoRoom 資料 | 否 |
| 4 | 一個訓練週期的試驗：量測顯示記憶體與時間 | 否，這是試驗 |
| 5 | 評估官方檢查點與隨機策略 | 否 |
| 6 | 檢查 Python 3.12 能否執行 | 否 |
| 7 | 把步驟 1 至 6 的結果交給工作階段，寫 0.1 的門檻鎖定紀錄 | 這一步就是鎖定 |
| 8 | 正式的重現訓練與評估 | 是，必須在標籤 `prereg/stage-0a` 建立之後 |

重現的環境選 TwoRoom：它是二維導航，資料與訓練成本在官方的四個環境中預期最小，也最接近格子世界。

## 前置條件

- Linux，或 Windows 上的 WSL2（Ubuntu）。官方指令以 bash 撰寫。
- 已安裝 NVIDIA 驅動，`nvidia-smi` 能看到 GPU。
- 已安裝 git、zstd、GNU time、uv：

```bash
sudo apt install -y git zstd time
curl -LsSf https://astral.sh/uv/install.sh | sh
```

- 磁碟空間：先在 Hugging Face 上查看 TwoRoom 資料壓縮檔的大小，預留解壓後與檢查點所需的空間。

## 步驟 1：記錄硬體與版本

```bash
mkdir -p ~/m1-results && cd ~/m1-results
nvidia-smi > gpu.txt
nvidia-smi --query-gpu=name,driver_version,memory.total --format=csv >> gpu.txt
uname -a > system.txt
lscpu | head -20 >> system.txt
free -h >> system.txt
```

## 步驟 2：安裝 LeWorldModel

```bash
cd ~
git clone https://github.com/lucas-maes/le-wm.git
cd le-wm
git rev-parse HEAD > ~/m1-results/lewm-commit.txt

uv venv --python=3.10
source .venv/bin/activate
uv pip install "stable-worldmodel[train,env]"
uv pip freeze > ~/m1-results/freeze-py310.txt
python -c "import torch; print(torch.__version__, torch.version.cuda, torch.cuda.get_device_name(0))" > ~/m1-results/torch.txt
```

## 步驟 3：下載 TwoRoom 資料

```bash
export STABLEWM_HOME=$HOME/stable-wm   # 放在空間足夠的磁碟
mkdir -p $STABLEWM_HOME
```

1. 開啟 Hugging Face 上的 LeWorldModel 合集：https://huggingface.co/collections/quentinll/lewm
2. 找到 TwoRoom 的資料集，下載其中的 `.tar.zst` 壓縮檔。
3. 解壓到 `$STABLEWM_HOME`：

```bash
tar --zstd -xvf <下載的檔名>.tar.zst -C $STABLEWM_HOME
ls $STABLEWM_HOME   # 應看到 tworoom.h5
```

之後每開一個新的終端機，都要重新 `export STABLEWM_HOME=$HOME/stable-wm` 並 `source ~/le-wm/.venv/bin/activate`。

## 顯示記憶體的量測方式

每次訓練或評估時，另開一個終端機執行，結束後按 Ctrl+C 停止：

```bash
nvidia-smi --query-gpu=timestamp,memory.used,memory.total,utilization.gpu --format=csv,noheader,nounits -l 1 > ~/m1-results/vram-<名稱>.csv
```

最高用量：

```bash
awk -F', ' '{ if ($2 > m) m = $2 } END { print m " MiB" }' ~/m1-results/vram-<名稱>.csv
```

這個數字包含 CUDA 本身的佔用，比 PyTorch 實際配置的量略高，是保守的上限。

## 步驟 4：一個訓練週期的試驗

先用官方預設設定（每批 128）跑一個訓練週期：

```bash
cd ~/le-wm
/usr/bin/time -v python train.py data=tworoom trainer.max_epochs=1 num_workers=4 \
  output_model_name=smoke_b128 subdir=smoke_b128 2>&1 | tee ~/m1-results/train-smoke_b128.log
```

- 若出現顯示記憶體不足，依序改為每批 64、32 再試，例如加上 `loader.batch_size=64`，並把 `output_model_name` 與 `subdir` 改成對應的名稱。
- `num_workers` 依 CPU 核心數調整。
- 每次都記錄：每批樣本數、是否成功、最高顯示記憶體、一個訓練週期的時間（記錄檔中 `Elapsed (wall clock) time` 一行）。

## 步驟 5：評估官方檢查點與隨機策略

```bash
cd ~/le-wm
python eval.py --config-name=tworoom.yaml policy=quentinll/lewm-tworooms world.max_episode_steps=100 \
  2>&1 | tee ~/m1-results/eval-official.log
python eval.py --config-name=tworoom.yaml policy=random world.max_episode_steps=100 \
  2>&1 | tee ~/m1-results/eval-random.log
```

- `world.max_episode_steps` 是評估設定中必填的值，必須不小於評估預算 50；這裡用 100，請照實記錄。
- 結果印在終端機的 `metrics` 一行，也會附加到 `$STABLEWM_HOME` 下對應目錄的 `tworoom_results.txt`。
- 若以 Hugging Face 名稱載入失敗，改用官方 README「From the Hugging Face mirror」一節的轉換方式，並記錄這件事。

## 步驟 6：檢查 Python 3.12

```bash
cd ~/le-wm
uv venv .venv312 --python=3.12
source .venv312/bin/activate
uv pip install "stable-worldmodel[train,env]"
python train.py data=tworoom trainer.max_epochs=1 num_workers=4 \
  output_model_name=smoke_py312 subdir=smoke_py312 2>&1 | tee ~/m1-results/train-smoke_py312.log
deactivate
```

能完成一個訓練週期就算可以執行；失敗時保留完整的錯誤訊息。

## 步驟 7：交付試驗結果

把下方的結果範本填好，連同 `~/m1-results/` 的檔案一起交給工作階段。工作階段會據此：

- 寫 0.1 的門檻鎖定紀錄：重現的容許範圍、種子數、使用的每批樣本數。
- 建立標籤 `prereg/stage-0a`。

另外請從 LeWorldModel 論文（arXiv:2603.19312）的結果表中，抄下 TwoRoom 的規劃成功率，以及論文回報的種子數與變異（例如平均 ± 標準差）。這是判定重現是否成功的參考值。

## 步驟 8：正式的重現訓練與評估

標籤 `prereg/stage-0a` 建立之後才執行。每批樣本數與種子數依鎖定紀錄。

```bash
cd ~/le-wm && source .venv/bin/activate
/usr/bin/time -v python train.py data=tworoom num_workers=4 seed=<鎖定紀錄中的種子> \
  output_model_name=tworoom_full_s<種子> subdir=tworoom_full_s<種子> [loader.batch_size=<鎖定的值>] \
  2>&1 | tee ~/m1-results/train-full-s<種子>.log

python eval.py --config-name=tworoom.yaml policy=tworoom_full_s<種子>/weights_epoch_100.pt world.max_episode_steps=100 \
  2>&1 | tee ~/m1-results/eval-full-s<種子>.log
```

- 檢查點存在 `$STABLEWM_HOME/checkpoints/<output_model_name>/`，評估時的路徑相對於 `$STABLEWM_HOME/checkpoints/`。
- 預設訓練 100 個週期。官方程式啟動時會在 `$STABLEWM_HOME/checkpoints/<subdir>/` 尋找 `<output_model_name>_weights.ckpt`，找到就從它繼續；若中斷，以相同指令重新執行，並記錄是否成功續跑。
- 訓練時同樣量測顯示記憶體。

## 結果範本

```
## M1 結果

- 日期：
- GPU 型號／顯示記憶體：
- 驅動／CUDA／PyTorch 版本：
- 作業系統（含是否為 WSL2）：
- LeWorldModel 提交：
- stable-worldmodel 版本：

### 步驟 4：一個訓練週期的試驗
| 每批樣本數 | 成功 | 最高顯示記憶體 (MiB) | 一個週期的時間 |
| --- | --- | --- | --- |
| 128 | | | |
| 64 | | | |
| 32 | | | |

### 步驟 5：評估
- 官方檢查點的 metrics：
- 隨機策略的 metrics：
- 評估時間：
- 是否需要改用 README 的轉換方式：

### 步驟 6：Python 3.12
- 是否完成一個訓練週期：
- 錯誤訊息（若有）：

### 論文的參考值
- TwoRoom 規劃成功率：
- 種子數與變異：
- 出處（表格編號）：

### 與手冊不符之處
-
```
