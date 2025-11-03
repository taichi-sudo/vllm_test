# 参考
https://zenn.dev/cloud_ace/articles/bc3492d5e5ab25

# 必要パッケージのインストール
## DockerおよびNVIDIA Container Toolkitのインストール（Ubuntu例）
```
sudo apt update
sudo apt install ca-certificates curl gnupg lsb-release
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io -y
```

## ファイアウォール設定
```
sudo ufw allow 5006/tcp
sudo ufw allow 11434/tcp
```

# コマンドを打ってollamaをインストール
```
curl -fsSL https://ollama.com/install.sh | sh
```

# Open WebUI のコンテナを起動
```
sudo docker run -d --network=host -v open-webui:/app/backend/data -e OLLAMA_BASE_URL=http://127.0.0.1:11434 --name open-webui --restart always ghcr.io/open-webui/open-webui:main
```

# API叩く
```

```
