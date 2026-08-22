# nginx-proxy/docker-gen オープン issue / PR 調査メモ

**調査日:** 2026-06-18
**対象リポジトリ:** https://github.com/nginx-proxy/docker-gen
**スコープ:** 当初 open issues 40件 / open PR 10件 を全件分析。**その後クローズされた issue / PR は本メモおよび併設 JSON から削除済み**（本メモは未解決のもののみを収録）。現在の収録数は open issue 15件 / open PR 9件。

**更新（2026-07-10）:** 前回 in-flight だった OPEN PR 3件がすべて upstream にマージされ、対応 issue がクローズされた（[#745](https://github.com/nginx-proxy/docker-gen/pull/745)→#238 / [#747](https://github.com/nginx-proxy/docker-gen/pull/747)→#452 / [#748](https://github.com/nginx-proxy/docker-gen/pull/748)→#227、いずれも MERGED）。close 待ちだった #252 / #280 もクローズ（COMPLETED）された。**これらマージ/クローズ済み項目は本メモから削除**。同日さらに #154 / #319 / #277 / #458 を実装し upstream へ提出（in-flight: [#755](https://github.com/nginx-proxy/docker-gen/pull/755)→#154 / [#756](https://github.com/nginx-proxy/docker-gen/pull/756)→#319 / [#757](https://github.com/nginx-proxy/docker-gen/pull/757)→#277 / [#758](https://github.com/nginx-proxy/docker-gen/pull/758)→#458、いずれも OPEN・CI green）。本メモは **open のまま残る項目** のみを収録。

**更新（2026-07-13）:** 回答のみで close 提案した #166 / #266 / #250 がクローズ（NOT_PLANNED）されたため本メモから削除。同日さらに、PR [#755](https://github.com/nginx-proxy/docker-gen/pull/755)(#154) / [#757](https://github.com/nginx-proxy/docker-gen/pull/757)(#277) がマージ（issue #154 CLOSED・旧 PR #277 も CLOSED）、close 提案した #172 / #296 / #297 / #226 / #179、および upstream #761 で修正された #628 もクローズ → **これらも本メモから削除**（#226・#179 は既存コメントと差別化して投稿）。残る in-flight は [#756](https://github.com/nginx-proxy/docker-gen/pull/756)(#319) / [#758](https://github.com/nginx-proxy/docker-gen/pull/758)(#458) / [#762](https://github.com/nginx-proxy/docker-gen/pull/762)(#187 lexists)（#758 は buchdag の指摘で `SetCurrentContainer` 方式へ再設計済み・OPEN）。

**更新（2026-07-28）:** PR [#758](https://github.com/nginx-proxy/docker-gen/pull/758)(#458) がマージされ、issue #458 がクローズ（COMPLETED）されたため本メモから削除。あわせて、**これまで初回調査時点の raw 記録として全件を保持していた併設 JSON 2ファイルからも、クローズ済み項目をすべて削除**した（issue 40件→15件 / PR 10件→9件。過去ラウンドで本メモからのみ削除していた 25件 + マージ済み PR [#277](https://github.com/nginx-proxy/docker-gen/pull/277) を含む）。これにより本メモと JSON の収録範囲が一致し、3ファイルとも「現在 open のもののみ」を表す。残る in-flight は [#756](https://github.com/nginx-proxy/docker-gen/pull/756)(#319) / [#762](https://github.com/nginx-proxy/docker-gen/pull/762)(#187 lexists)（いずれも OPEN）。

**目的:** 本ドキュメントは、当ダウンストリーム fork のメンテナが、upstream の未解決 issue / 古い PR を **自分の手で段階的に実装・解消していく** ための恒久的な調査メモである。upstream へ PR を提出することは目的ではなく、現行ローカルコードベース（`internal/` 再編後・Go 1.25.5・`fsouza/go-dockerclient` v1.13.2）に対して各項目を再評価し、「そのまま使えるか / リベースが必要か / 作り直すべきか / 無視すべきか」を判断するための材料を残すことを目的とする。

**併設データ:** 各項目の生（raw）の構造化分析（要約・難易度・key files・rationale など）を、本メモと同じディレクトリに JSON で保存している。

- [`open-prs-analysis-data.json`](open-prs-analysis-data.json) — オープン PR 9件の項目別分析
- [`open-issues-analysis-data.json`](open-issues-analysis-data.json) — オープン issue 15件の項目別分析

> 注: JSON の各エントリは項目ごとの個別調査エージェントの生出力のままであり（クローズ済み項目の削除以外の加筆修正はしていない）、一部に依存 API 可用性の誤評価（後述「検証メモ」で訂正した #718 / #240 の「コンパイルする」という記述）を含む。食い違う場合は本 Markdown を正とする。

---

## コメントで close できそうな / 対応不要な open 項目

open だが実装不要の候補。**C=PR**、**D=検証で未解決・close しない**（クローズ済みの項目は本メモから削除。旧 A「コメントで close 可」と B「前提消滅/仕様通り」の候補はすべてクローズ済みのため節ごと削除。#267 は feature request と判明したため swarm クラスタへ移動）。

### C. PR（コメントして close 候補）

| PR | 対応 |
|---|---|
| [#462](https://github.com/nginx-proxy/docker-gen/pull/462) | Go 変更なしの低価値 example。**close 推奨** |
| [#247](https://github.com/nginx-proxy/docker-gen/pull/247) | 依存の Swarm 管理 API が消滅。`-notify-filter label=com.docker.swarm.service.name=<svc>` で代替可。close 候補 |

### D. 検証で「未解決」と判明 → close しない（残課題）

2026-06-21 の反証検証で、当初 A 候補に挙げたが **実際には現行コードでも再現し得る** と判明。close せず、修正候補として扱う。

| # | 残課題（file:line 根拠） |
|---|---|
| [#269](https://github.com/nginx-proxy/docker-gen/issues/269) | go-dockerclient の再接続上限（5回 / backoff）は `eventHijack` が **接続エラーを返したときのみ** 適用。issue 前提「デーモンはソケットを受けるが未初期化」では `eventHijack` が nil を返し上限をバイパス → reader goroutine が非EOFデコードエラーで spin → `disableEventMonitoring` を経ず `monitorEvents` が即再接続で **sleep なしの /events 連打**。docker-gen の10秒 backoff（`generator.go:263/289`）は発火しない。要 docker-gen 側ガード（再接続の最小間隔 / レート制限） |
| [#201](https://github.com/nginx-proxy/docker-gen/issues/201) | "is a directory"（ユーザ起因）と SIGHUP race は解決済み。だが `signal.Ignore` は **SIGHUP のみ**（`main.go:168`）で、報告症状 signal 2=SIGINT は無防備。起動時の同期 `generateFromContainers()`（`generator.go:82`）が SIGINT/SIGTERM ハンドラ登録（`:83-85`）**前**に走る窓で kill-signal-2 が今も起こり得る。初期生成前に INT/TERM ハンドラを登録すれば解消 |

> 注: close コメント文面は投稿前に要確認。

---

## 概要 / エグゼクティブサマリ

### 全体傾向

- **古い項目の多くは現行コードで既に解決済み、または前提が消滅して陳腐化している。** 特に 2015〜2018 年の issue は、`internal/` への再編・モダンな `fsouza/go-dockerclient` への移行・signal handling の書き直しによって、実コードを変更しなくても解消しているものが目立つ。
- **Swarm 関連は一大クラスタだが、いずれも「やる価値が薄い」側。** PR #247 / #548 / #718、issue #207 / #240 / #267 がすべて Swarm（mode / classic）に関係する。現行の pinned 依存 `fsouza/go-dockerclient` v1.13.2 は Swarm の task/service API を実質的に落としており（`misc.go` には "Docker Swarm is deprecated" の注記）、業界全体の Swarm 離れもあって、ほとんどが low priority。
- **本当に「動く、得する」機会は少数の easy / additive な変更に集中している。**

### 検証メモ（2026-06-18 実機確認・重要訂正）

個別調査の一部に依存 API の可用性について食い違いがあったため、ローカルで `go build` により決着させた。

- **`fsouza/go-dockerclient` v1.13.2 には Swarm の service/task 系 API（`ListServices` / `ListServicesOptions` / `ListTasks` / `ListSwarmNodes` / `InspectService` 等）が存在しない。** `c.ListServices(...)` を呼ぶ最小ファイルは `c.ListServices undefined (type *docker.Client has no field or method ListServices)` でコンパイル失敗。client には該当する `func (c *Client) ...Service/Task/Swarm/Node...` メソッドが1つもなく、`container.go`/`event.go`/`misc.go` に残る "Swarm" は legacy classic Swarm（`container.Node` / `SwarmNode`）と deprecation 注記のみ。
- 帰結:
  - **PR [#718](https://github.com/nginx-proxy/docker-gen/pull/718)（Add swarm service）は現状の pinned client ではそのままビルドできない。** `getServices()` が依存する `c.ListServices()` が無いため、`easy / rebase-and-finish` ではなく **`hard`**。実装には go-dockerclient のアップグレード（swarm 対応版）か公式 Docker SDK（`github.com/docker/docker/client`）の併用が前提。
  - #207 / #247 の「Swarm 管理 API は（pinned client では）使えない」という評価が正しい。#240 / #718 の個別調査にあった「`svc_apicheck_tmp.go` でコンパイルする / 型チェック済み」という記述は本検証で **否定**された（以降の該当箇所は本メモを正とする）。
  - PR [#548](https://github.com/nginx-proxy/docker-gen/pull/548) は `ListContainers` を複数エンドポイントで回すだけで swarm service API に依存しないため、この訂正の影響を受けない（medium のまま）。

### 推奨：最初に着手すべき TOP 候補

| 種別 | # | 内容 | 理由 |
|---|---|---|---|
| PR | **#713** | exec によるコンテナ内コマンド通知 | MERGEABLE、現行レイアウトで書かれている。call site を2か所足すだけで完成。実用的な機能ギャップを埋める |

### 「やらない」と判断すべき主なもの

- Swarm 系（#207 / #240 / #267 / PR #247 / #548 のクラスタ）は、fork が明確に multi-node swarm を狙わない限り skip。

---

## コードベース概要

future reader 向けの Go コード配置マップ：

```
docker-gen/
├── cmd/docker-gen/
│   └── main.go              CLI エントリポイント。フラグ定義(initFlags)、TOML設定読込
│                            (loadConfig)、GeneratorConfig 組み立て、NewGenerator→Generate()
├── internal/
│   ├── config/config.go     Config 構造体（Template/Dest/NotifyCmd/NotifyContainers/
│   │                        ContainerFilter/Interval/Wait など）、ParseWait
│   ├── context/
│   │   ├── context.go       テンプレートに渡るデータモデル。Context(=[]*RuntimeContainer)、
│   │   │                    RuntimeContainer、Network、State、Health、DockerImage、
│   │   │                    SwarmNode(legacy)。Env()/Docker() メソッド、CurrentContainerID 検出
│   │   └── address.go       GetContainerAddresses / renderAddress（ポート→Address 変換）
│   ├── dockerclient/
│   │   └── docker_cli.go    NewDockerClient / GetEndpoint / SplitDockerImage
│   ├── generator/
│   │   └── generator.go     中核。NewGenerator、getContainers、generateFromContainers /
│   │                        generateAtInterval / generateFromEvents、newDebounceChannel、
│   │                        runNotifyCmd / sendSignalToContainers、signal handling
│   ├── template/
│   │   ├── template.go      newTemplate(FuncMap 登録, sprig 統合)、GenerateFile、executeTemplate
│   │   ├── where.go         where / whereNot / whereLabel* 系
│   │   ├── groupby.go       groupBy / groupByMulti / generalizedGroupByKey
│   │   ├── sort.go          sortObjectsByKeys* など
│   │   ├── functions.go     include / exists など
│   │   └── reflect.go       deepGet 等
│   └── utils/utils.go       PathExists、SplitKeyValueSlice など
├── templates/               配布サンプル: nginx.tmpl, etcd.tmpl, fluentd.conf.tmpl,
│                            logrotate.tmpl, dnsmasq.hosts.conf.tmpl
├── app/docker-entrypoint.sh コンテナ ENTRYPOINT（exec "$@"）
├── Dockerfile.alpine / Dockerfile.debian
└── go.mod                   Go 1.25.5, fsouza/go-dockerclient v1.13.2, sprig v3.3.0,
                             BurntSushi/toml v1.6.0
```

> 注: 調査中に Swarm API 可用性を確かめるため `svc_apicheck_tmp.go` という untracked スクラッチファイルを一時生成し、`go build` で検証後に削除した（結果は冒頭「検証メモ」を参照）。`go.sum` への副次変更も revert 済みで、本コミットには調査メモ（`docs/investigation/`）のみを含める。

---

## オープン PR セクション

### サマリ表（優先度順）

| # | タイトル | 状態 | 難易度 | 推奨アクション | 優先度 |
|---|---|---|---|---|---|
| [#713](https://github.com/nginx-proxy/docker-gen/pull/713) | feat: Adding new Notify Option using exec | mergeable | easy | rebase-and-finish | **medium** |
| [#548](https://github.com/nginx-proxy/docker-gen/pull/548) | pull data from multiple docker endpoints (swarm) | conflicting | hard | reimplement | **medium** |
| [#718](https://github.com/nginx-proxy/docker-gen/pull/718) | Add swarm service | mergeable(※ビルド不可) | hard | reimplement | low |
| [#319](https://github.com/nginx-proxy/docker-gen/pull/319) | splitKeyValuePairs / groupByMultiKeyValuePairs | conflicting | easy | 🛠 PR #756 オープン | low |
| [#202](https://github.com/nginx-proxy/docker-gen/pull/202) | Handle kill event | obsolete | medium | reimplement | low |
| [#247](https://github.com/nginx-proxy/docker-gen/pull/247) | Add support for notifying a service | obsolete | medium | reimplement | low |
| [#254](https://github.com/nginx-proxy/docker-gen/pull/254) | pongo2 as alternative to Go Template | obsolete | hard | reimplement | low |
| [#596](https://github.com/nginx-proxy/docker-gen/pull/596) | feat: Add WASM module templates | conflicting | hard | needs-discussion | low |
| [#462](https://github.com/nginx-proxy/docker-gen/pull/462) | Trivial schema/user creation example | mergeable | trivial | close | skip |

### PR 詳細

#### [#713](https://github.com/nginx-proxy/docker-gen/pull/713) feat: Adding new Notify Option using exec — **最優先候補**

- **概要:** テンプレート再生成後に、対象コンテナ内で Docker exec API（CreateExec/StartExec/InspectExec）経由でコマンドを実行する `NotifyContainersCmd` config オプション（コンテナ名/ID → argv の map）を追加。既存の signal ベース通知（NotifyContainers / NotifyContainersFilter）を補完する。動機例は `pihole reloaddns`。
- **状態/コンフリクト:** **非コンフリクト。** GitHub は MERGEABLE（mergeStateStatus=BLOCKED は CI/レビューゲートであってコード競合ではない）。2025-11-26 の新しい PR で現行 `internal/` レイアウトに対して書かれており、`internal/config/config.go:16-19` の挿入点も generator のアンカー（notify-call ブロック、`sendSignalToFilteredContainers`）も完全一致。diff はクリーンに当たる。
- **難易度/工数:** easy / 約1-2h（採用 + 欠けた call site 追加 + テスト）。
- **推奨アクション:** rebase-and-finish。
- **key files:** `internal/config/config.go`, `internal/generator/generator.go`, `cmd/docker-gen/main.go`, `README.md`
- **rationale:** 実コードに本物のギャップを埋める。現状 docker-gen はコンテナに signal を送る（`sendSignalToContainer`/`KillContainer`）かホスト側 shell コマンド実行（`runNotifyCmd`）しかできず、**別コンテナ内でコマンドを実行する手段がない**。実装は小さく idiomatic で、go-dockerclient の安定 exec API を使用。設定配線も無料（TOML が `config.Config` へ直接デコードされる）。**ただし要修正の incompleteness:** `g.sendCmdToContainers(cfg)` が `generateFromEvents`（generator.go:~218）にしか配線されていない。既存の通知機構はすべて `generateFromContainers`（:137-139）と `generateAtInterval`（:168-170）からも呼ばれており、このままだと初回/起動時生成と interval 生成では新コマンドが発火しない。欠けた2つの call site を足すのは trivial。マイナーな nit: `Tty:true`+`RawTerminal:true` と非スレッドセーフな `bytes.Buffer` の出力キャプチャはやや奇妙。テストなし。

#### [#548](https://github.com/nginx-proxy/docker-gen/pull/548) Add support to pull data from multiple docker endpoints (e.g. swarm)

- **概要:** 反復指定可能な `-swarm-node` フラグを追加し、複数の Docker API エンドポイント（swarm node ごと）からコンテナ/ネットワークのメタデータを集約する。NewGenerator が node ごとに `*docker.Client` を構築し、generateFromEvents をエンドポイントごとの event-watcher goroutine に書き換え、getContainers を全 swarm client でループ。制御アクション（sendSignal/SetServerInfo/SetDockerEnv）は意図的にローカル `g.Client` に残す。著者は本番運用中でクリーンに書き直す意思あり。
- **状態/コンフリクト:** CONFLICTING / DIRTY。テキスト競合だけでなく**意味的衝突**が2つ。(1) `getContainers` は `func (g *generator) getContainers(config config.Config)` に変わり、`config.ContainerFilter` を `ListContainers` に渡して包含を駆動する（generator.go:391-403）。PR は削除済みの `g.All` フィールドと per-config フィルタなしで getContainers を全書き換えするため、採用すると container-filter / include-stopped / only-exposed 機能を退行させる。(2) `generator`/`GeneratorConfig` 構造体から `All` フィールド自体が消えているため**そもそもコンパイルしない**。generateFromEvents の書き換えは concurrency-sensitive（unbuffered な clientDone/done チャネルを複数 goroutine から書く）で手作業の再適用が必要。
- **難易度/工数:** hard / 約数日。
- **推奨アクション:** reimplement（リベース不可、現行 generator.go に対し書き直し）。
- **key files:** `internal/generator/generator.go`, `internal/generator/generator_test.go`, `cmd/docker-gen/main.go`, `internal/dockerclient/docker_cli.go`, `README.md`
- **rationale:** 機能アイデアは依然有効で、著者の中核設計（制御アクションはローカル維持しつつ read-only データを複数エンドポイントから集約）は現行コードでも成立する。支援部品（`dockerclient.GetEndpoint`/`NewDockerClient`、main.go のフラグ配線）は揃っている。Swarm クラスタの中では唯一 priority medium だが、これは「複数エンドポイント集約」という概念が container-filter 機能と直交していて筋が良いため。著者が書き直しを申し出ている点がコストを下げる。

#### [#718](https://github.com/nginx-proxy/docker-gen/pull/718) Add swarm service

- **概要:** Docker Swarm の *service* 列挙を追加し、テンプレートが（コンテナだけでなく）サービスを iterate できるようにする。`RuntimeService` 型 + パッケージレベル store + `Service()`/`SetServices()`（context.go）、`Client.ListServices()` を呼ぶ `getServices()`（generator.go）、3つの生成パスへの配線、デモ `templates/services.tmpl` を追加。`.Service` でアクセス。+120/-3、3ファイル、単一コミット。
- **状態/コンフリクト:** diff のテキスト適用は一致（2026-01-09 の新しい PR で現行 post-reorg レイアウトを対象、context.go の var ブロック / Docker()/SetServerInfo()、generator.go の getContainers()/3生成ループの diff context が一致）。**ただし致命的:** PR が依存する `c.ListServices()` / `docker.ListServicesOptions` / `swarm.Service` は vendored go-dockerclient v1.13.2 に**存在せず、そのままではコンパイルしない**（2026-06-18 に `go build` で `c.ListServices undefined` を確認、上の「検証メモ」参照）。GitHub の MERGEABLE はテキスト merge 可否であってビルド可否ではない点に注意。
- **難易度/工数:** hard / 約数日（go-dockerclient のアップグレード or 公式 Docker SDK の併用が前提）。
- **推奨アクション:** reimplement（テキストだけ当てても compile しないため、swarm 対応 client の導入から設計し直す）。
- **key files:** `internal/context/context.go`, `internal/generator/generator.go`, `internal/template/template.go`, `templates/services.tmpl`, `go.mod`, `go.sum`
- **rationale:** 現行コードに Swarm service サポートは皆無（legacy `SwarmNode` のみ）なので真に新規。**最大の障壁は依存:** pinned client が swarm service/task API を持たないため、まず client 戦略（アップグレード or 公式 SDK 併用）を決める必要がある。それを除いても **機能が未完成**: `g.generateFromServices()` が Generate() 内でコメントアウト（generator.go ~82）、未使用の `Service struct{Name string}`、opt-in フラグ/設定がなく `getServices()` が**非 Swarm デーモンでも毎サイクル ListServices() を実行**しエラー時に return するため**非 Swarm ユーザーのコンテナ生成を中断する退行**になる。テスト/README/まともなテンプレートもなし。priority は low（#548 と統合検討の余地あり、Swarm クラスタ参照）。

#### [#319](https://github.com/nginx-proxy/docker-gen/pull/319) two new functions splitKeyValuePairs and groupByMultiKeyValuePairs — 🛠 PR #756 オープン

- **概要:** 2つのテンプレートヘルパーを追加。`splitKeyValuePairs($string,$listSep,$kvpSep,[$defaultKey])` は文字列を map[string]string にパース。`groupByMultiKeyValuePairs` は groupByMulti 同様だがフィールド値を key/value にパースしキーでグループ化。README/単体テスト付き。用途例: `VIRTUAL_PORT="443:3000,3000:3000"` のような env のパース。
- **状態/コンフリクト:** CONFLICTING。2020年の PR で旧モノリシック root-level `template.go`/`template_test.go` を編集しており、これらは存在しない。groupBy 系は `internal/template/groupby.go`、FuncMap 登録は `template.go`(newTemplate)、テストは `groupby_test.go` に再編済み。FuncMap ブロックは完全に再構成され（sprig チェーン + 構造体リテラル）、PR のカラム整列 diff は完全にノイズ。手作業で再適用必須。
- **難易度/工数:** easy / 約1-2h。
- **推奨アクション:** reimplement。
- **key files:** `internal/template/groupby.go`, `internal/template/template.go`, `internal/template/groupby_test.go`, `README.md`
- **rationale:** 両関数とも現行コードに同等物なし（groupByMulti は値を plain string に分割するだけ、sprig にも kv-grouping ヘルパーなし）。再実装は小さく機械的（既存 `generalizedGroupByKey` を再利用）。**移植時に直すべき品質問題が2つ:** (1) PR は `strings.Split(kvp, kvpSep)` で `[0]`/`[1]` を index しており、値にセパレータが含まれると余分なセグメントを黙って落とす → `strings.SplitN(...,2)` を使う。(2) PR のテストはエントリがポインタなのに値型 `RuntimeContainer` にキャスト（`groups["445"][1].(RuntimeContainer)`）しており現行コードでは panic する。priority low（niche な利便ヘルパー、関連 issue なし）。

#### [#202](https://github.com/nginx-proxy/docker-gen/pull/202) Handle kill event

- **概要:** Docker が `kill` イベント（`docker stop` で発火）を出したとき true になる `Killing` bool を container `State` に追加し、graceful-shutdown ドレイン中はコンテナを「既に消えた」扱いにできるようにする。動機: nginx-proxy のロードバランシングで、shutdown 中のコンテナを即座に生成 config から外す（新規リクエスト停止）一方、既存接続は SIGTERM 完了まで維持する。
- **状態/コンフリクト:** obsolete。触れていた3ファイル（context.go/generator.go/template.go）はすべて root-level で消滅。さらに配線点がすべて消えている: (1) per-config consumer ループ（generator.go:206）は今や `for range debouncedChan` でイベントを無視（API 再 list するだけ）。(2) `newDebounceChannel` が多数イベントを1つに collapse し per-event identity を破棄。(3) PR が改修した event-status whitelist は消滅し現在は全イベントを fan-out。(4) `filterRunning` は消滅、`Running` は set されるがどのテンプレートからも読まれない。
- **難易度/工数:** medium / 約半日。
- **推奨アクション:** reimplement。
- **key files:** `internal/context/context.go`, `internal/generator/generator.go`, `internal/template/template.go`, `internal/template/where.go`
- **rationale:** リテラル diff はリベース不可。`Killing bool` の追加自体は trivial だが、現行 consumer ループが個別イベントを見ず、kill state は Docker API 再 list から復元不能（`docker inspect` に "killing" 概念なし）。忠実な再実装は event-fanout goroutine から per-config consumer へ per-container kill state を通し、getContainers() 再 list を生き延びさせる必要があり元の ~15行 patch より相当重い。さらに `Running` のテンプレート側 consumer が今は無いため、kill 中コンテナのテンプレートへの surface 方法も決める必要あり。価値は niche（nginx-proxy の graceful-drain）、2016年から未応答で low priority。

#### [#247](https://github.com/nginx-proxy/docker-gen/pull/247) Add support for notifying a service

- **概要:** `-service-notify-sighup <serviceID>` フラグと `NotifyServices map[string]docker.Signal` を追加し、Swarm サービスを支える全コンテナを HUP する。`ListTasks` で task を列挙し各 task の `Status.ContainerStatus.ContainerID` を SIGHUP。`SetServerInfo` を `*docker.Env`→`*docker.DockerInfo` に移行する付随変更も含む。
- **状態/コンフリクト:** obsolete。テキスト競合より深い問題: (1) **依存先の Swarm 管理 API が消滅** — `fsouza/go-dockerclient` v1.13.2 には `ListTasks`/`Task`/`ContainerStatus` が無い（モジュールキャッシュを grep して確認、`SwarmNode` と deprecated な `SwarmInfo` のみ残存）。`g.Client.ListTasks(...)` はコンパイルしない。(2) `SetServerInfo` の型変更は既に upstream で完了済み（redundant）。(3) config 形状が `NotifyContainers map[string]int` に変わり PR のパターンと不一致。(4) main.go/generator.go が移動・大幅リファクタ。
- **難易度/工数:** medium（swarm ラベル経由で再実装なら約半日 / swarm 対応クライアント再導入なら数日）。
- **推奨アクション:** reimplement。
- **key files:** `internal/generator/generator.go`, `internal/config/config.go`, `cmd/docker-gen/main.go`, `internal/context/context.go`, `go.mod`
- **rationale:** 重要: **現行コードは既に別の動く機構でほぼ同等の価値を提供している。** `sendSignalToFilteredContainers` + `-notify-filter` フラグで label/name フィルタによる signal が可能で、swarm task はラベル `com.docker.swarm.service.name` を持つため、local-node ケースなら `-notify-filter label=com.docker.swarm.service.name=<svc>` でコード変更なしに実現できる。専用フラグは ergonomics のみ。cross-node 列挙は swarm 対応クライアントの再導入（大規模、upstream は swarm を deprecate）が必要。2017年の空ボディ PR、6年放置。

#### [#254](https://github.com/nginx-proxy/docker-gen/pull/254) Add ability to use pongo2 as an alternative to Go Template

- **概要:** pongo2（Django/Jinja2 風 Go テンプレートエンジン）を `-engine` フラグ（"go" デフォルト / "pongo2"）または TOML `engine = "pongo2"` で選択可能な代替として追加。`pongoContext()` ヘルパーで docker-gen のテンプレート関数を pongo2.Context にマッピングし、サンプルテンプレートを pongo2 構文に移植。著者は関心測定の proposal でテスト/docs 未記述と明言。
- **状態/コンフリクト:** obsolete。2017年の flat layout / GLOCKFILE+glock を対象で全 hunk が失敗。さらに意味的にも stale: 現行 funcmap は大きく分岐（sprig 統合 + sprig* エイリアス、eval/include closure、fromYaml/toYaml、whereNot、groupBy*WithDefault、sort* 系、comment、toLower/toUpper）しており PR の hardcode map には無い。`Context.Env`/`Context.Docker` は今やメソッド。executeTemplate は `;`-split の複数ファイル ParseFiles に対応するが pongo branch は非対応。
- **難易度/工数:** hard / 約数日。
- **推奨アクション:** reimplement（具体的なダウンストリーム需要がある場合のみ。なければ close）。
- **key files:** `internal/template/template.go`, `internal/config/config.go`, `cmd/docker-gen/main.go`, `internal/context/context.go`, `internal/template/where.go`, `internal/template/groupby.go`, `go.mod`
- **rationale:** diff リベース不可。実質「ゼロからの新機能」で、しかも**第2のテンプレート言語を恒久的に parity 維持する大きなメンテ負担** + 新依存（flosch/pongo2）。jwilder は2017年に "I like it" と言ったがテスト/マージせず放置。fork には Jinja2 構文が hard requirement でない限りコスト/ベネフィットが悪い。関連 issue #177。

#### [#596](https://github.com/nginx-proxy/docker-gen/pull/596) feat: Add WASM module templates

- **概要:** コンパイル済み WebAssembly(WASI) モジュールを「テンプレート」として使えるようにする。テンプレートパスが `.wasm` で終わる場合、`tetratelabs/wazero` で実行し、container context を JSON 化して stdin へ渡し stdout を出力とする。WASM 作者が import する公開 `plugin` Go モジュール（別 go.mod）、easyjson 生成コード、`-wasmcache` フラグ、go.work ワークスペースを追加。
- **状態/コンフリクト:** CONFLICTING / DIRTY。統合 seam は小さい（`GenerateFile` で `filepath.Ext(config.Template)==".wasm"` 分岐するだけ）が周辺がすべて stale: go.mod は 1.21→1.25.5 で依存刷新、PR の go.mod/go.sum/go.work 追加は obsolete で再生成必須。`GenerateFile` の hunk が想定する filteredContainers ブロックは存在しない。**重要: context 構造体に drift はなく** wasm.go の converter は現行 context 型でコンパイルする。
- **難易度/工数:** hard / 約数日。
- **推奨アクション:** needs-discussion。
- **key files:** `internal/template/template.go`, `internal/context/context.go`, `cmd/docker-gen/main.go`, `go.mod`, `go.sum`, `README.md`
- **rationale:** HARD なのは seam ではなく周辺: (1) 依存/ビルド変更が完全に stale で go.work/easyjson を一から再生成。(2) **versioned な公開 `plugin/` モジュール（別 go.mod）+ 1385行の easyjson 生成コード + internal/context と永久に手動同期が必要な複製構造体** — 真のメンテ負担でメンテナが渋る最大要因。(3) バイナリ `.wasm` を examples/ に commit（CI で tinygo ビルドすべき）。(4) `log.Fatalf`/`log.Panicf` でエラー処理（executeTemplate の構造と不整合）。2024年2月、レビューコメントゼロ、関連 issue なし。WASM プラグインが実際に欲しいかで価値が決まる。欲しい場合は stale な multi-module/easyjson 機構をリベースするより wazero 呼び出し側を現行コードに再実装する方がクリーン。

#### [#462](https://github.com/nginx-proxy/docker-gen/pull/462) Trivial implementation of automated schema/user creation

- **概要:** `templates/` 配下に新規3ファイル（mariadb.tmpl / mariadb.dockerfile / mariadb-entrypoint.sh）を追加。DB_HOST/DB_USER/DB_PASS/DB_SCHEMA env を露出するコンテナごとに `CREATE USER`/`CREATE SCHEMA`/`GRANT` の mysql コマンドを emit する例。Go コードは変更なし。著者自身が "just good enough for dev" と述べている。
- **状態/コンフリクト:** MERGEABLE（28行追加・0削除、Go ソース不変）。`range $index,$value := .` と `$value.Env.DB_HOST` の構文は現行 `nginx.tmpl` と同一で今のエンジンでもレンダ可能。
- **難易度/工数:** trivial / 採用なら約30分（主に cleanup）、close なら0。
- **推奨アクション:** **close**。
- **key files:** `templates/nginx.tmpl`, `templates/etcd.tmpl`, `README.md`, `internal/template/template.go`, `internal/context/context.go`
- **rationale:** 低価値の example/docs 寄稿で機能追加なし。fork に得るものは実質なし（アイデアはコード変更なしで既に動く）。品質問題: `mariadb.dockerfile` は abandoned な `jwilder/docker-gen` を FROM、`mariadb-entrypoint.sh` は sleep/backoff なしの無限 `while true`、パスワード hardcode（`-ppass`）+ TODO。README/テスト未配線で orphan。欲しければ綺麗に書き起こす方が速い。

---

## オープン issue セクション

### サマリ表（カテゴリ別）

#### バグ / correctness

| # | タイトル | カテゴリ | 妥当性 | 難易度 | 優先度 |
|---|---|---|---|---|---|
| [#269](https://github.com/nginx-proxy/docker-gen/issues/269) | Excessive /events calls | bug | 🛠 still-relevant（残課題・D 参照） | unknown | **要修正** |
| [#201](https://github.com/nginx-proxy/docker-gen/issues/201) | "is a directory" / killed signal 2 | bug | 🛠 partially（残課題・D 参照） | trivial | **要修正** |

#### template-function / context 拡張

| # | タイトル | カテゴリ | 妥当性 | 難易度 | 優先度 |
|---|---|---|---|---|---|
| [#330](https://github.com/nginx-proxy/docker-gen/issues/330) | Add image label info | feature-request | still-relevant | easy | low |
| [#151](https://github.com/nginx-proxy/docker-gen/issues/151) | filter containers reachable by docker-gen | template-function | partially-addressed | easy | low |
| [#174](https://github.com/nginx-proxy/docker-gen/issues/174) | static vs dynamic HostPort判定 | feature-request | still-relevant | easy | low |
| [#185](https://github.com/nginx-proxy/docker-gen/issues/185) | reverse IP addr 関数 | template-function | still-relevant | easy | low |
| [#187](https://github.com/nginx-proxy/docker-gen/issues/187) | Add lexists function | template-function | 🛠 PR #762 オープン | trivial | low |

#### 機能要求 / 運用

| # | タイトル | カテゴリ | 妥当性 | 難易度 | 優先度 |
|---|---|---|---|---|---|
| [#67](https://github.com/nginx-proxy/docker-gen/issues/67) | Add check option | feature-request | still-relevant | easy | low |
| [#131](https://github.com/nginx-proxy/docker-gen/issues/131) | use as a library | feature-request | partially-addressed | medium | low |
| [#197](https://github.com/nginx-proxy/docker-gen/issues/197) | change user permission of output | feature-request | still-relevant | easy | low |
| [#246](https://github.com/nginx-proxy/docker-gen/issues/246) | Verbose logging | feature-request | still-relevant | easy | low |
| [#180](https://github.com/nginx-proxy/docker-gen/issues/180) | Template from URL | feature-request | still-relevant | medium | low |

#### Swarm / Docker API

| # | タイトル | カテゴリ | 妥当性 | 難易度 | 優先度 |
|---|---|---|---|---|---|
| [#207](https://github.com/nginx-proxy/docker-gen/issues/207) | Docker 1.12 Swarm Mode | feature-request | still-relevant | hard | low |
| [#240](https://github.com/nginx-proxy/docker-gen/issues/240) | How about swarm? (again) | feature-request | still-relevant | hard | low |
| [#267](https://github.com/nginx-proxy/docker-gen/issues/267) | remote node detection over swarm | feature-request | still-relevant | hard | low |

### issue 詳細（still-relevant — 低優先・概要のみ）

- **[#330](https://github.com/nginx-proxy/docker-gen/issues/330) Add image label info（easy/low）:** コンテナの underlying image labels（Dockerfile LABEL）をテンプレートに公開。`DockerImage` に `Labels map[string]string` 追加 + getContainers で `InspectImage` 呼び出し。**image ID で結果をキャッシュ**しないと N コンテナで毎サイクル N API 呼び出しの perf 退行。exposed-ports サブ要求は skip。caddy-gen/nginx-proxy の「image が自前 defaults を ship」UX を狙う場合のみ。関連 #303, #239。
- **[#185](https://github.com/nginx-proxy/docker-gen/issues/185) reverse IP addr 関数（easy/low）:** Part 1 = `reverseDns(ip)`（in-addr.arpa / ip6.arpa）を `functions.go` に追加（約30-60分、自己完結）。Part 2（`-interval` をテンプレートに公開し DNS TTL に）はより invasive で marginal。Part 1 のみ推奨。
- **[#187](https://github.com/nginx-proxy/docker-gen/issues/187) Add lexists function（trivial/low, 🛠 PR #762 オープン）:** `os.Lstat` ベースの `utils.PathLExists` を `exists` の lstat 版として追加、`lexists` で登録。symlink 自体（dangling 含む）を検出。約15行。
- **[#174](https://github.com/nginx-proxy/docker-gen/issues/174) static vs dynamic HostPort（easy/low）:** `HostConfig.PortBindings` を参照し、要求された host port が空かで static/dynamic を判定。`Address` に `HostPortStatic bool`（推奨はより情報量の多い `RequestedHostPort string`）追加 + `GetContainerAddresses` に HostConfig を渡す。具体テンプレート需要がある場合のみ。
- **[#151](https://github.com/nginx-proxy/docker-gen/issues/151)（easy/low）:** docker-gen と同一ネットワークを共有するコンテナのみに絞る `whereSharedNetwork` ヘルパー。`CurrentContainerID` + `Networks` で実装、`-event-filter event=connect/disconnect` で reactivity。generator/context 変更不要。関連 #149。
- **[#67](https://github.com/nginx-proxy/docker-gen/issues/67)（easy/low）:** notify 前に走る check コマンド（例 `nginx -t` 成功時のみ reload）。file-level 解釈のみ実装推奨: `CheckCmd string` 追加 + `-check` フラグ + `runNotifyCmd` 内でゲート（3 call site を1変更でカバー）。per-container 版は skip。`nginx -t && nginx -s reload` で既に表現可能なため low。
- **[#197](https://github.com/nginx-proxy/docker-gen/issues/197)（easy/low）:** 生成ファイルの owner/group/mode 制御。`os.WriteFile(..., 0644)` がハードコード。`Mode string`（mode-only から）+ 任意で `Owner`/`Group` を追加。`os.Chown` は Windows で no-op なので build tag / `runtime.GOOS` ガード必須。notifyCmd での chown 回避策が既存で low。
- **[#246](https://github.com/nginx-proxy/docker-gen/issues/246)（easy/low）:** verbose/debug ロギング。標準 `log` のみでレベルなし。`-verbose` + `DEBUG` env を追加、`log/slog` 推奨。container discovery のデバッグ行（どのコンテナが list/除外されたか）が最も価値。default 挙動は不変に保つ。
- **[#180](https://github.com/nginx-proxy/docker-gen/issues/180)（medium/low）:** URL からテンプレート取得。`resolveTemplatePath` ヘルパーで http(s) scheme を検出し起動時に1回 fetch して temp ファイルへ。毎イベント fetch は remote を叩きすぎるので fetch-once-at-startup 推奨。idiomatic にはマウント/イメージ焼き込みで足りるため low。関連 #192。
- **[#131](https://github.com/nginx-proxy/docker-gen/issues/131)（medium/low, partially-addressed）:** ライブラリ利用。`GeneratorConfig`/`NewGenerator` 構造は既にあるが、全パッケージが `internal/` 下で外部 import 不可、`NewGenerator` が unexported `*generator` を返す。最小パス: パッケージを `internal/` 外へ昇格 + `generator` 型を export。public API 契約を永久に背負う + upstream との恒久 merge friction に注意。ライブラリ embedding が具体要件の場合のみ。関連 #130。

### issue 詳細（already-fixed / obsolete — 簡潔メモ）

- **[#201](https://github.com/nginx-proxy/docker-gen/issues/201)（一部のみ・残課題、2026-06-21 訂正）:** "is a directory" は bind-mount ユーザーエラー、SIGHUP race は `signal.Ignore(syscall.SIGHUP)`（main.go:168）+ graceful handler + entrypoint `exec "$@"` で解決済み。**ただし報告症状 signal 2=SIGINT は未解決**: 起動時の同期 `generateFromContainers()`（generator.go:82）が INT/TERM ハンドラ登録（:83-85）前に走る窓で ungraceful 終了が起こり得る（→ セクション D）。close しない。
- **[#269](https://github.com/nginx-proxy/docker-gen/issues/269)（残課題、2026-06-21 訂正）:** /events 過剰呼び出し。**当初 already-fixed と評価したが反証で覆った**: デーモンがソケットを受けるが未初期化のとき `eventHijack` が nil を返し go-dockerclient の再接続上限がバイパスされ、sleep なしで /events を連打し得る。docker-gen の10秒 backoff は発火しない（→ セクション D）。close せず docker-gen 側に再接続ガードが必要。

### issue 詳細（Swarm クラスタ — low/skip）

- **[#207](https://github.com/nginx-proxy/docker-gen/issues/207) Docker 1.12 Swarm Mode（hard/low）/ [#240](https://github.com/nginx-proxy/docker-gen/issues/240) How about swarm? again（hard/low）:** docker-gen に swarm-mode awareness は皆無（既存 `SwarmNode`/`container.Node` は legacy classic Swarm 用）。**依存可用性は実機検証で決着済み（冒頭「検証メモ」参照）: pinned go-dockerclient v1.13.2 は swarm service/task API（`ListServices`/`ListTasks`/`ListSwarmNodes` 等）を露出しておらず、`c.ListServices` 等は `go build` でコンパイル失敗する。** したがって #207 の「API を落としている」評価が正しく、#240 個別調査にあった「v1.13.2 で `svc_apicheck_tmp.go` がコンパイルする」は誤り。swarm を進めるには **client のアップグレードか公式 Docker SDK 併用が前提**で、実コストはそこに加えて「設計」（service/task context モデル、template data-model の後方互換、VIP vs per-task IP semantics、swarm event は /events に来ないため interval polling）で multi-day〜weeks。fork が multi-node swarm を明確に狙わない限り skip。関連 #239, #303。
- **[#267](https://github.com/nginx-proxy/docker-gen/issues/267) remote node detection over swarm（hard/low, still-relevant, 2026-07-13 訂正で B→feature request に移動）:** multi-node swarm で他ノードのコンテナも見たいという要望。**close しない feature request**（2018〜2024 に interest 継続。candeemis の実装案「ノード列挙 → 各ノードで `ListContainers` → マージ」を nabheet が 2024 に支持）。ノード**自動**列挙は swarm-mode API（pinned client に無い）依存だが、**PR #548 方式（ユーザーが各ノードのエンドポイントを渡す複数エンドポイント集約）なら swarm API 不要で実現可能**。fork が multi-node swarm を狙う場合に #548 と統合して対応。回避策（per-node に docker-gen 1台）は現状動く。

---

## 優先度付き実装ロードマップ

### フェーズ 1 — Quick wins（trivial〜easy、純加算 / 低リスク）

順に着手推奨：

1. **PR #713 exec notify を仕上げ**（easy、1-2h）— **最実用的。** MERGEABLE な PR を採用し、欠けた2 call site（generateFromContainers / generateAtInterval）を追加 + スモークテスト。

### フェーズ 2 — 任意の便利関数 / QoL（easy、需要ベース）

- **#185 reverse-DNS 関数**（Part 1 のみ）、**#187 lexists（🛠 PR #762）**、**#151 whereSharedNetwork**、**#174 HostPort static/dynamic**、**#67 check option**、**#197 file mode**、**#246 verbose logging**、**#330 image labels**（キャッシュ必須）。
- これらは fork の具体ニーズが出たときに opportunistic に。

### フェーズ 3 — 大規模 / 慎重判断（hard、需要が明確な場合のみ）

- **Swarm クラスタ（PR #548 / #718 + issue #207 / #240 / #267 / #247）:** multi-node swarm が fork の core 用途でない限り skip。**前提条件は検証済み（冒頭「検証メモ」）: pinned go-dockerclient v1.13.2 に swarm service/task API は無く、`ListServices` 等はコンパイルしない。** したがって service 列挙系（PR #718 / issue #207 / #240 / #247）は client のアップグレードか公式 Docker SDK 併用が必須で hard。やるなら、API 非依存で筋の良い **PR #548（複数エンドポイントで `ListContainers` を集約、medium）を先に**、service 列挙（PR #718）は client 戦略を決めてから。
- **PR #596 WASM**（needs-discussion）/ **PR #254 pongo2**（reimplement）/ **#131 library 化**（medium）/ **#180 template from URL**（medium）: 恒久メンテ負担 or 公開 API 契約を伴う。具体的ダウンストリーム需要がある場合のみ。

### 着手しない（close 候補）

- **PR #462**（低価値 example）、**PR #202 / #247**（依存 API 消滅 or 既存機構で代替可）。
- issue（回答 close 候補）: すべて投稿・クローズ済み。
- **#201 / #269 は close 候補から除外**（2026-06-21 検証で未解決と判明・残課題 → 「コメントで close できそうな open 項目」節 D）。

---

## 付録: 全項目フラットリスト（1行ステータス）

### オープン PR（9件）

- **#713** feat: exec notify — mergeable / easy / **rebase-and-finish** / medium。call site 2か所追加で完成、最実用的。
- **#548** multiple docker endpoints (swarm) — conflicting / hard / reimplement / medium。getContainers が container-filter と意味的衝突、書き直し必須。著者が再実装意思あり。
- **#718** Add swarm service — **ビルド不可** / hard / reimplement / low。依存 `c.ListServices` が pinned go-dockerclient v1.13.2 に無くコンパイル不可（検証メモ参照）。加えて未完成（commented out generateFromServices、非 Swarm で退行）。
- **#319** splitKeyValuePairs/groupByMultiKeyValuePairs — conflicting / easy / reimplement / low。SplitN と pointer cast を修正して移植。🛠 PR #756 オープン。
- **#202** Handle kill event — obsolete / medium / reimplement / low。配線点が全消滅、kill state は再 list で復元不能。
- **#247** notify a service — obsolete / medium / reimplement / low。依存の Swarm API 消滅。`-notify-filter label=...` で代替可。
- **#254** pongo2 engine — obsolete / hard / reimplement / low。実質新機能 + 第2言語の恒久メンテ負担。
- **#596** WASM templates — conflicting / hard / needs-discussion / low。別 plugin module + easyjson の恒久同期負担。
- **#462** mariadb example — mergeable / trivial / **close** / skip。低価値 example、品質問題あり。

### オープン issue（15件）

- **#67** check option — still-relevant / easy / low。file-level のみ実装、`nginx -t && reload` で代替可。
- **#131** use as a library — partially-addressed / medium / low。GeneratorConfig はあるが internal/ で import 不可。
- **#151** filter reachable containers — partially-addressed / easy / low。whereSharedNetwork ヘルパー追加。
- **#174** static/dynamic HostPort — still-relevant / easy / low。PortBindings で判定。
- **#180** Template from URL — still-relevant / medium / low。起動時1回 fetch。
- **#185** reverse IP 関数 — still-relevant / easy / low。Part 1（reverseDns）のみ。
- **#187** lexists 関数 — trivial / low。os.Lstat 版。🛠 PR #762 オープン。
- **#197** output permission — still-relevant / easy / low。Mode 追加、Chown は Windows ガード。
- **#201** is-a-directory/signal 2 — 🛠 **残課題（2026-06-21 検証訂正）**。"is a directory"/SIGHUP は解決済みだが signal 2=SIGINT は初期同期生成の窓で未解決（D 参照）。close しない。
- **#207** Docker 1.12 Swarm Mode — still-relevant / hard / low。swarm クラスタ、設計重い。
- **#240** swarm again — still-relevant / hard / low。swarm クラスタ。**検証済み: v1.13.2 に swarm service/task API は無い**ため client アップグレード/公式 SDK が前提。
- **#246** verbose logging — still-relevant / easy / low。-verbose + slog、discovery デバッグ行。
- **#267** remote node over swarm — still-relevant / hard / low。swarm cluster の feature request（2024 まで実装要望）。#548 方式（複数エンドポイント集約）で実現可能。fork が multi-node swarm を狙う場合のみ。
- **#269** excessive /events — 🛠 **残課題（2026-06-21 検証訂正）**。デーモンがソケットを受けるが未初期化のとき go-dockerclient の再接続上限がバイパスされ /events 連打が再現し得る（D 参照）。close しない。
- **#330** image label info — still-relevant / easy / low。InspectImage、image ID キャッシュ必須。
