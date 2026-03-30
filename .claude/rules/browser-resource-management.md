# ブラウザリソース管理ルール

## 第一原則: ブラウザリソースには上限がある

サーバーサイドの感覚で「作って使って捨てる」を繰り返してはいけない。
ブラウザのランタイムリソースには個数・メモリの上限があり、超過すると既存リソースが停止・破棄される。

## リソース別の上限と対策

| リソース | 上限目安 | 超過時の挙動 | 対策 |
|---------|---------|-------------|------|
| AudioContext | 6〜8個（ブラウザ依存） | 古いコンテキストがsuspend → マイク等が死亡 | **1つを再利用**。毎回 new しない |
| WebSocket | ブラウザ/ドメインあたり数十 | 接続拒否 | 不要な接続は即 close |
| MediaStream | デバイスごとに1〜数個 | 取得失敗 or 前のストリーム停止 | 使用後は tracks.forEach(t => t.stop()) |
| Web Worker | 数十（メモリ依存） | メモリ不足 | 不要なWorkerは terminate() |
| BroadcastChannel | 制限なし（メモリ依存） | メモリリーク | 使用後は close() |

## 必須パターン: リソースの再利用

```javascript
// Bad: 毎回作成（AudioContext枯渇の直接原因）
function playAudio(data) {
  const ctx = new AudioContext({ sampleRate: 24000 });
  // ... use ctx ...
  // ctx.close() しても上限カウントは即座にリセットされない
}

// Good: 1つを再利用
let playCtx = null;
function playAudio(data) {
  if (!playCtx || playCtx.state === "closed") {
    playCtx = new AudioContext({ sampleRate: 24000 });
  }
  // ... use playCtx ...
}
```

## 必須パターン: クリーンアップ順序（末端→根元）

リソースを閉じる順序は「末端から根元へ」。依存されている側を先に閉じると、依存している側が孤児になる。

```
1. タイマー        clearTimeout / clearInterval
2. 再生キュー      queue = []; isPlaying = false;
3. 子リソース      stopRecording() — MediaStream tracks 停止、ScriptNode 切断
4. コンテキスト    audioCtx.close(); playCtx.close();
5. 接続            ws.close();
6. UI状態          DOM更新、フラグリセット
```

### 禁止パターン
```javascript
// Bad: audioCtx を先に閉じると、stopRecording() 内の scriptNode.disconnect() が失敗する可能性
audioCtx.close();
stopRecording();  // 孤児リソースが残る

// Good: 子を先に停止してから親を閉じる
stopRecording();
audioCtx.close();
```

## 必須パターン: 全リソースの棚卸し

リアルタイム通信機能の設計時、以下のチェックリストで使用するブラウザリソースを全て洗い出す:

| # | 確認項目 | 変数名 | 作成タイミング | 破棄タイミング | 再利用するか |
|---|---------|--------|-------------|-------------|------------|
| 1 | WebSocket | ws | start() | goBack() / エラー時 | No（毎回新規） |
| 2 | 録音AudioContext | audioCtx | start() | goBack() | No（毎回新規） |
| 3 | 再生AudioContext | playCtx | playNext() | goBack() | Yes（セッション中再利用） |
| 4 | MediaStream | mediaStream | getUserMedia() | stopRecording() | No |
| 5 | ScriptNode | scriptNode | createScriptProcessor() | stopRecording() | No |
| 6 | タイマー | lateTranslateTimer | onmessage | goBack() / ws.onclose | N/A |

この表を設計段階で作成し、「破棄タイミング」に漏れがないことを確認する。

## マルチラウンドテスト義務

リソースの作成→使用→破棄→再作成のサイクルは、**最低3ラウンド**テストする。

1回目で動いても、2回目以降で枯渇・リーク・状態不整合が発生する。
「1回OK」は「テストしていない」と同義。

**ローカル環境でリアルサーバー/API/デバイスが使えない場合でも、モックを作ってフルサイクルを検証する。**
「ローカルで再現できないからテストできない」は禁止。詳細: `.claude/rules/development-workflow.md` の「リソースライフサイクルのモック検証」セクション。

テスト項目（各ラウンド終了後に検証）:
- [ ] WebSocket: `ws === null`
- [ ] AudioContext: `audioCtx === null` (or closed)
- [ ] PlaybackContext: `playCtx === null` (or closed)
- [ ] MediaStream: `isRecording === false`, tracks stopped
- [ ] タイマー: 全てクリア済み
- [ ] DOM: 前セッションの残骸なし
- [ ] 再開始: 上記が全てクリーンな状態から新規セッションが正常に開始できる

## API 呼び出しコンテキストの検証

### ブラウザ API には呼び出し元の前提条件がある

ブラウザ API の呼び出し元を変更する（ユーザージェスチャー → コールバック、同期 → 非同期等）場合、呼び出し先 API の前提条件が壊れていないか検証する。

| API | 前提条件 | 違反時の挙動 |
|-----|---------|-------------|
| getUserMedia | ユーザージェスチャー（クリック等）のコールスタック内 | 権限拒否 or 無応答 |
| Notification.requestPermission | ユーザージェスチャー | 自動拒否 |
| window.open | ユーザージェスチャー | ポップアップブロック |
| AudioContext (new) | ユーザージェスチャー（一部ブラウザ） | suspended 状態で作成 |
| Clipboard API (write) | ユーザージェスチャー | 権限拒否 |

### 検証タイミング
- **設計時**: API 呼び出しの移動・変更がある場合、呼び出し元のコンテキストが要件を満たすか確認
- **コードレビュー時**: 関数の呼び出し元が変わった差分を検出したら、前提条件チェックを必須とする
- **テスト時**: プレビュー環境は権限が緩い場合がある。実ブラウザでの検証を怠らない

### 発生事例（2026-03-29a）
EchoTalk2 で getUserMedia をユーザージェスチャー（ボタンクリック）から WebSocket コールバックに移動。プレビュー環境では権限が緩く検出できず、実ブラウザでのみマイクが動作しなくなった。

## 発生事例（2026-03-29b）

EchoTalk2で `playNext()` が毎回 `new AudioContext()` を作成。1回目の会話は正常だが、音声再生のたびにAudioContextが累積し、2回目の会話でブラウザ上限に到達。録音用AudioContextがsuspendされ、マイクが一切反応しなくなった。修正: 再生用AudioContextを `playCtx` 変数で保持し再利用。3ラウンドテストで累積バグがないことを検証。
