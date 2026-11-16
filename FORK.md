# この fork の運用

`numachang/sendo-reader` は、`sendo-feed/sendo-reader` を Cloudflare にデプロイして使うための fork。
上流の開発用の PR・レビュー・マージ手順とは別に、更新を自動で取り込む。

## 上流への自動追従

上流の `main` に変更が入ると、数分のうちに
[Sync upstream](https://github.com/numachang/sendo-reader/actions/workflows/sync-upstream.yml) が、
上流の `main` をこの fork の `main` に取り込む。GitHub の「Sync fork」と同じ API を使い、
独自コミットを残す。force push はせず、競合した場合は失敗して停止し、上書きしない。
取り込みで `main` が更新されると、接続済みの Cloudflare Workers Builds がビルドとデプロイを行う。

Sync upstream を起動するのは、Cloudflare の Worker
[fork-sync-cron](https://github.com/numachang/fork-sync-cron)。毎分、上流との差分を調べ、
遅れていれば workflow_dispatch で起動する。GitHub Actions の `schedule` は使わない
（定刻の実行が作られないことがあったため。経緯は fork-sync-cron の README）。

reader は、サーバー側の `numachang/sendo` が上流に追いつき、そのデプロイ（`Workers Builds: sendo`）が
成功してから取り込む。API の追加など順序が必要な更新でも、手で順番を守らなくてよい。

- 手動で動かすには、Actions の「Sync upstream」から「Run workflow」を選ぶ（ブランチは `main`）。
  手動のときは、サーバー側を待たない。
- 取り込みの認証には、自動発行される `GITHUB_TOKEN`（`contents: write`）を使う。
  Worker が起動に使うトークンは、fork-sync-cron の README を見る。
- このワークフローは `numachang/sendo-reader` でだけ動く。
- 同時実行は直列にし、実行中の同期を取り消さない。
- 自動の同期を止めるには、Actions の「Sync upstream」から「Disable workflow」を選ぶ。
  Worker は無効なワークフローを起動しない。
- 失敗したら Actions のログを確認する。競合は手動で解消してから、もう一度動かす。
  Worker は、上流に新しいコミットが来るか1時間たつまで、失敗した同期を起動し直さない。
- 上流が進んでいるのに同期が起動されなければ、fork-sync-cron の README の「失敗したら」を見る。
  サーバー側のデプロイが失敗している間も、reader は待つ。
- 同期後のデプロイは、同期先コミットの「Workers Builds: sendo-reader」と Cloudflare のビルド履歴で確認する。

参照：[fork の同期 API](https://docs.github.com/en/rest/branches/branches#sync-a-fork-branch-with-the-upstream-repository)、
[Workers Builds](https://developers.cloudflare.com/workers/ci-cd/builds/)、
[起動する Worker](https://github.com/numachang/fork-sync-cron)、
[サーバー側の運用](https://github.com/numachang/sendo/blob/main/FORK.md)。
