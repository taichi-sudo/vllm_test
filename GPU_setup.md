
# 必要パッケージのインストール
## DockerおよびNVIDIA Container Toolkitのインストール（Ubuntu例）
```
sudo apt update
sudo apt install -y docker.io
# NVIDIA Container Toolkit導入（NVIDIA GPUを使う場合）
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt update
sudo apt install -y nvidia-docker2
sudo systemctl restart docker

sudo systemctl start docker
sudo systemctl enable docker
```

## ファイアウォール設定
```
sudo ufw allow 5006/tcp
sudo ufw allow 11434/tcp
```
## GPU対応 OllamaサーバーDockerコンテナ起動
```
sudo docker run -d --gpus=all -p 11434:11434 --name ollama-server ollama/ollama
```

##モデル（例：llama2:7b）ダウンロード
・モデルを追加したいときは下記のモデル名を変更
```
sudo docker run -d --gpus=all -p 11434:11434 --name ollama-server ollama/ollama
```

## GPU対応 Open WebUIコンテナの起動（OllamaAPI連携）
```
sudo docker run -d -p 5006:8080 --gpus all --name openwebui --restart always \
-e OLLAMA_BASE_URL="http://ollama-server:11434" \
--network bridge \
ghcr.io/open-webui/open-webui:cuda
```

# 動作確認（Dockerログ/サービス状態）
## コンテナ状態確認
```
sudo docker ps
```

## ログ確認（エラーや起動状態監視）
```
sudo docker logs ollama-server
sudo docker logs openwebui
```

## ブラウザアクセス
```
http://[GCE外部IP]:5006/
```

## API叩くなら
```
curl http://localhost:11434/v1/chat/completions \
    -H "Content-Type: application/json" \
    -d '{
        "model": "gemma3:4b",
        "messages": [
            {
                "role": "user",
                "content": "ChatGPTを開発している企業を答えてください。"
            }
        ]
    }'

```
