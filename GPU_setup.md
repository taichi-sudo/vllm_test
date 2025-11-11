# 参考
https://zenn.dev/cloud_ace/articles/bc3492d5e5ab25

# 必要パッケージのインストール
## DockerおよびNVIDIA Container Toolkitのインストール（Ubuntu例）
```
sudo apt update
sudo apt install ca-certificates curl gnupg lsb-release -y
sudo mkdir -p /etc/apt/keyrings

# ファイルに一旦保存してからdearmor実行
curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o docker.gpg
sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg docker.gpg
rm docker.gpg

sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io -y
```

## ファイアウォール設定
```
sudo ufw allow 11434/tcp
```

# コマンドを打ってollamaをインストール
```
curl -fsSL https://ollama.com/install.sh | sh
```

---
★★★★★★以下が表示されたときの対応★★★★★★
```
ERROR: NVIDIA GPU detected, but your OS and Architecture are not supported by NVIDIA. Please install the CUDA driver manually https://docs.nvidia.com/cuda/cuda-installation-guide-linux/
```
NVIDIAのCUDAドライバを自分でいれろっているエラー。
まずは依存パッケージのインストール。
```
sudo apt-get update
sudo apt-get install build-essential dkms linux-headers-$(uname -r) curl -y
```
ここ（https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64 ）からドライバインストール手順を進める。
deb(ネットワーク)を選択すると以下のような表示になるので、GCE等の操作対象のコンソールで実行。
```
# CUDA toolkit
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt-get update
sudo apt-get -y install cuda-toolkit-13-0

# NVIDIA ドライバー
sudo apt-get install -y nvidia-open
sudo apt-get install -y cuda-drivers
```
nvidiaのコンテナtoolkitをinstallする
```
sudo apt-get update
sudo apt-get install --reinstall nvidia-container-toolkit
```
※完了したらまたOllamaをインストールする。

★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★
---

使いたいモデルを下記小マントでインストールする

```
ollama run deepseek-r1:70b
```
# Open WebUI のコンテナを起動
```
sudo systemctl start docker
sudo docker run -d --network=host -v open-webui:/app/backend/data -e OLLAMA_BASE_URL=http://127.0.0.1:11434 --name open-webui --restart always ghcr.io/open-webui/open-webui:main
```

GCEファイアウォールルールの設定をする。11434など、指定したポートを通すルールを追加する。
続いては、Ollamaの設定ファイルを作成する。

nginxのインストール
```
sudo apt update
sudo apt install nginx-core
```

アクセストークンがあるリクエストのみパス
```
sudo vim /etc/nginx/sites-available/ollama
```

```
# -----------------------------------------------------------
# アクセストークンを定義する
# "Bearer [トークン]" 1; という形式で記述します。
# -----------------------------------------------------------
map $http_authorization $valid_token {
    "Bearer xxxxxxxxxxxxxxxxxxxxx" 1; 

    # トークン追加後は以下に記載してく
    # "Bearer xxxxxxxxxxxxxx" 1;
    default 0;
}

server {
    # 11434番ポートで外部からのリクエストを待機
    listen 11434;
    listen [::]:11434;
    
    # サーバーのIPアドレス
    server_name xxx.xxx.xxx.xxx; 

    location / {
        # -----------------------------------------------------------
        # 認証チェック
        # $valid_token が 0 (無効) だった場合、
        # 401 Unauthorized (認証エラー) 
        # -----------------------------------------------------------
        if ($valid_token = 0) {
            return 401 '{"error": "Unauthorized"}';
        }

        # -----------------------------------------------------------
        # 認証が通れば、内部(localhost)で動いているOllamaにリクエストを中継
        # -----------------------------------------------------------
        proxy_pass http://127.0.0.1:11434; # 内部のOllama
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Ollamaのストリーミング応答に対応するための設定
        proxy_buffering off;
        proxy_cache off;
        proxy_set_header Connection '';
        proxy_http_version 1.1;
        chunked_transfer_encoding off;
    }
}
```

設定ファイルのシンボリックリンクを作成（sites-enabledに登録）：
```
sudo ln -s /etc/nginx/sites-available/ollama /etc/nginx/sites-enabled/
```
nginx設定ファイルの文法チェック：
```
sudo nginx -t
```

nginxのリロード（設定反映）：
```
sudo systemctl reload nginx
```

設定ファイル用のディレクトリを作成
```
sudo mkdir -p /etc/systemd/system/ollama.service.d/
```

設定ファイル (override.conf) を直接作成 （OLLAMA_HOST=0.m.m.m をファイルに書き込みます）
```
echo -e "[Service]\nEnvironment=\"OLLAMA_HOST=0.0.0.0\"" | sudo tee /etc/systemd/system/ollama.service.d/override.conf
```
設定のリロード
```
sudo systemctl daemon-reload
sudo systemctl restart ollama
sudo ss -tlnp | grep 11434
```

API叩いてテスト
```
curl http://localhost:11434/api/generate \
  -H "Authorization: Bearer xxxxxxxxxxxxxxxx" \
  -d '{
    "model": "deepseek-r1:70b",
    "prompt": "Why is the sky blue?",
    "stream": false
  }'
```
