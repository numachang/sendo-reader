# この fork の運用

`numachang/sendo-reader` は、`sendo-feed/sendo-reader` を Cloudflare にデプロイして使うための fork。
上流の開発用の PR・レビュー・マージ手順とは別に、更新を自動で取り込む。

## 上流への自動追従

[sendo-updater](https://github.com/sendo-feed/sendo-updater)（fork は `numachang/sendo-updater`、
Cloudflare の Worker は `sendo-updater`）が、毎分この fork と上流の差分を調べ、遅れていれば
上流の `main` をこの fork の `main` に取り込む（「Sync fork」と同じ API）。独自コミットは残り、
競合すれば上書きせずに止まる。取り込んだ push で、Workers Builds がビルドとデプロイを行う。

reader は、`numachang/sendo` が上流に追いつき、そのデプロイ（`Workers Builds: sendo`）が成功してから
取り込む。API の追加など順序が必要な更新でも、手で順番を守らなくてよい。

- 仕組み、トークン、困ったときの直し方は sendo-updater の README を見る。決定は、sendo の
  `docs/decisions/deploy.md` の 7 と、この repo の `docs/decisions/deploy.md`。
- 手で取り込むときは、fork の画面で「Sync fork」→「Update branch」を押す。sendo を先にする。
- 自動の取り込みを止めるには、Cloudflare で `sendo-updater` の Worker を消す（sendo の fork の取り込みも止まる）。
- 競合したら、`sendo-updater` のログに `failed (409)` が出る。fork で手で直す。
- 同期後のデプロイは、同期先コミットの「Workers Builds: sendo-reader」と Cloudflare のビルド履歴で確認する。

参照：[sendo-updater](https://github.com/sendo-feed/sendo-updater)、
[Workers Builds](https://developers.cloudflare.com/workers/ci-cd/builds/)、
[サーバー側の運用](https://github.com/numachang/sendo/blob/main/FORK.md)。
