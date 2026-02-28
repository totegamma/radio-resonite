# Radio Resonite

Resoniteからプレイヤーの緯度経度座標をWebSocketで受信し、[radio-browser.info](https://www.radio-browser.info/) APIで最寄りのラジオ局を検索してストリームURLを返すサーバー。

## 必要なもの

- Node.js v18以上

## セットアップ

```bash
git clone https://github.com/rabbuttz/radio-resonite.git
cd radio-resonite
npm install
```

## 起動

```bash
npm start
```

デフォルトポートは `3210`。変更する場合:

```bash
PORT=8080 npm start
```

## サーバーで常駐化する場合

### pm2を使う方法

```bash
npm install -g pm2
pm2 start npx --name radio-resonite -- tsx src/index.ts
pm2 save
pm2 startup
```

主なコマンド:

```bash
pm2 logs radio-resonite   # ログを見る
pm2 restart radio-resonite # 再起動
pm2 stop radio-resonite    # 停止
```

### systemdを使う方法 (Linux)

`/etc/systemd/system/radio-resonite.service` を作成:

```ini
[Unit]
Description=Radio Resonite
After=network.target

[Service]
Type=simple
WorkingDirectory=/path/to/radio-resonite
ExecStart=/usr/bin/npx tsx src/index.ts
Restart=always
Environment=PORT=3210

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl enable radio-resonite
sudo systemctl start radio-resonite
```

## WebSocketプロトコル

### Resonite → Server

座標を送信:

```json
{ "type": "update_position", "lat": 35.6762, "lon": 139.6503 }
```

### Server → Resonite

```json
{
  "type": "station",
  "station": {
    "name": "Station Name",
    "url": "http://stream.example.com/radio",
    "codec": "MP3",
    "bitrate": 128,
    "country": "Japan",
    "lat": 35.68000,
    "lon": 139.65030
  }
}
```

## 仕組み

1. 起動時にradio-browser.info APIから全局データをキャッシュ
2. WebSocket接続を受け付ける
3. 座標データ受信時、キャッシュから最寄り局を検索して即座に返す
