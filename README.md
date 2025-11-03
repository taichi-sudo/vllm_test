# 全体イメージ
* Ollama：複数のLLMモデル（例：Llama 3、Gemmaなど）をローカルやサーバー上でインストールし、APIサーバー（ポート11434）として動作。CLIでも操作可能だが、基本的にモデルの実行基盤。
* Open WebUI：Dockerコンテナとして動作し、ブラウザでアクセス可能なチャットUIを提供。ユーザーはOpen WebUIに対話的に接続し、プロンプト送信や会話履歴管理、ファイルアップロードなどが可能。
* 連携方法：Open WebUIは起動時、環境変数OLLAMA_BASE_URLとしてOllamaのAPIエンドポイント（例：http://ollama-server:11434）を指定し、このAPIを通じてモデル推論を依頼・取得する。内部的にはHTTP経由でリクエスト・レスポンスをやり取り。
* ネットワーク構成：両者は同じDockerネットワーク（bridge）内に置かれるか、Dockerホストのポートを介して疎通可能。これによりOpen WebUIのフロントエンドからOllamaのバックエンドAPIが使われる。
* UX：Ollama単独だとコマンドライン操作になるが、Open WebUIを組み合わせることでChatGPT風のGUIが提供され、モデル切替やチャット履歴保存、検索拡張生成（RAG）などの高度機能も利用可能になる。


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

## APIを叩くなら
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
