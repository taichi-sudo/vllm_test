# vllm_test

# 必要パッケージのインストール
## DOcker

```
sudo apt update
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
```

## ファイアウォール
```
sudo ufw allow 5006/tcp
sudo ufw allow 11434/tcp
```

##  Open WebUIおよびOllamaサーバーをDockerで起動
```
sudo docker run -d -p 11434:11434 --name ollama-server ollama/ollama
```

## モデル（例：llama2:7b）ダウンロード
・モデルを追加したいときは下記のモデル名を変更
```
sudo docker exec -it ollama-server ollama pull llama2:7b
```

## Open WebUIコンテナの起動（OllamaAPIとの連携）
```
sudo docker run -d -p 5006:8080 --name openwebui --restart always \
-e OLLAMA_BASE_URL="http://ollama-server:11434" \
--network bridge \
ghcr.io/open-webui/open-webui:main
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

