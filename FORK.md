# この fork の運用

`numachang/sendo-reader` は、`sendo-feed/sendo-reader` を Cloudflare にデプロイして使うための fork。
上流の開発用の PR・レビュー・マージ手順とは別に、更新を自動で取り込む。

## 上流への自動追従

[Sync upstream](https://github.com/numachang/sendo-reader/actions/workflows/sync-upstream.yml) が毎時27分に、
上流の `main` をこの fork の `main` に取り込む。GitHub の「Sync fork」と同じ API を使い、
独自コミットを残す。force push はせず、競合した場合は失敗して停止し、上書きしない。
更新がなければコミットは増えない。
取り込みで `main` が更新されると、接続済みの Cloudflare Workers Builds がビルドとデプロイを行う。
GitHub Actions から Cloudflare を直接デプロイする設定や追加のトークンは不要。

- 手動で動かすには、Actions の「Sync upstream」から「Run workflow」を選ぶ（ブランチは `main`）。
- 認証には自動発行される `GITHUB_TOKEN`（`contents: write`）を使う。追加のシークレットは不要。
- このワークフローは `numachang/sendo-reader` でだけ動く。
- 同時実行は直列にし、実行中の同期を取り消さない。
- 停止するには、Actions の「Sync upstream」から「Disable workflow」を選ぶ。
- 失敗したら Actions のログを確認する。競合は手動で解消してから、もう一度動かす。
- 同期後のデプロイは、同期先コミットの「Workers Builds: sendo-reader」と Cloudflare のビルド履歴で確認する。

サーバー側の `numachang/sendo` は毎時17分、reader は毎時27分に同期する。
これはサーバーを先に更新するための時間差だが、定期実行の遅延やビルド時間によって順序は保証されない。
API の追加など順序が必要な更新では、先に sendo のデプロイ成功を確かめてから reader を手動同期する。

GitHub の定期実行には遅延があり、公開リポジトリで60日間活動がないと自動で無効になる。
その場合は Actions から有効に戻す。

参照：[fork の同期 API](https://docs.github.com/en/rest/branches/branches#sync-a-fork-branch-with-the-upstream-repository)、
[定期実行の制約](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows#schedule)、
[Workers Builds](https://developers.cloudflare.com/workers/ci-cd/builds/)、
[サーバー側の運用](https://github.com/numachang/sendo/blob/main/FORK.md)。
