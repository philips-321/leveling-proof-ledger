# Repository Operating Rules

These instructions apply to every coding agent working in this repository.

Preserve the existing project, privacy, deployment and source-of-truth boundaries. Read relevant project documentation before changing behavior.

<!-- BEGIN SHARED ACTIONS BUDGET POLICY v1 -->
## Catatan kerja dan anggaran Actions (berlaku lintas repo)

Aturan ini berlaku untuk setiap agen. Pertahankan aturan khusus proyek di atas. Kebijakan lengkap: `philips-321/master-task-ledger:operations/actions-budget/POLICY.md` (privat). Jika tidak dapat membacanya, aturan minimum di bawah tetap berlaku; jangan mengklaim telah membaca sumber yang tidak tersedia.

- Git menyimpan JSON/Markdown tanpa memerlukan GitHub Actions. Jangan membuat workflow untuk sekadar menyimpan catatan, heartbeat, lock, atau membangun laporan atas catatan yang tidak berubah.
- Simpan hasil/checkpoint penting segera secara durable ke repo/path proyek yang sesuai; jangan menahan hasil hanya di workspace sementara sampai jadwal sinkronisasi. Jangan meletakkan data pribadi di repo publik. Local-only, committed, deployed, dan verified adalah status berbeda.
- Master ledger tetap sumber koordinasi lintas proyek. Baca task/project dan claim kanonis terkait, bukan hanya cache `state/*`. Claim tulis harus terlihat sebelum mutasi yang berpotensi tumpang tindih dan dilepas saat selesai. Jangan menunda claim demi batching.
- Gabungkan hasil + catatan proyek + handoff dalam satu commit per milestone bila memungkinkan. Update master pada mulai/perubahan lingkup, milestone bermakna, blocker baru, atau handoff/selesai; tidak setiap tool call. Run tanpa hasil baru cukup satu ringkasan bila status berubah; jangan menulis ulang nilai identik. Jangan menggabungkan pekerjaan berbeda ke satu task.
- Catatan rutin di JSON/MD, AGENTS, dan snapshot anggaran tidak boleh memicu CI, deploy, remote D1, semantic provisioning, atau scan seluruh repo. Cache boleh diperbarui terjadwal secara batch; cache bukan otoritas lock atau bukti hasil. Request JSON khusus yang sengaja menjalankan pekerjaan adalah pengecualian dan wajib melalui preflight biaya.
- Sebelum push/PR/merge, rerun, dispatch, atau menambah jadwal, baca pemicu workflow dan hitung job yang akan terpicu, termasuk downstream `workflow_run`, matrix, duplikasi push+PR, retries, serta commit otomatis. Jika tindakan ini akan menjalankan Actions, baca snapshot anggaran akun terbaru dan reservasi bersama di master ledger.
- Kuota akun dibagi semua repo privat, bukan 2.000 menit per repo. Hitung per job: `ceil(durasi_detik / 60)`, lalu jumlahkan semua job dan percobaan. Job paralel tetap dijumlah; gagal/dibatalkan setelah mulai tidak gratis. Perkiraan wall-clock workflow bukan perkiraan kuota.
- Laporkan sebelum eksekusi berbiaya: sumber/waktu snapshot, pemakaian resmi jika tersedia, rekonstruksi terpisah, reservasi belum terselesaikan, estimasi tambahan job ini, dan sisa konservatif. Angka yang tidak tersedia wajib `unknown`/`null`; jangan mengarang sisa aktual atau menyebut estimasi sebagai billing resmi. Snapshot lama/bulan berbeda tidak boleh dipakai sebagai izin.
- Estimasi recurring wajib mencakup sebulan penuh serta job turunannya, bukan satu run. Gunakan riwayat durasi konservatif dan timeout; jangan rerun tanpa perubahan atau diagnosis baru. Maksimal satu retry otomatis untuk gangguan sementara.
- Default internal akun: 2.000 menit included, batas rutin 1.500, cadangan 500; peringatan mulai 1.200. Pada proyeksi >=1.500 hentikan pekerjaan nonpenting; >=1.800 pause otomatisasi nonpenting. Jangan aktifkan overage/paid plan atau menghabiskan cadangan tanpa instruksi eksplisit pemilik. Ini batas internal, bukan aturan produk GitHub.
- Jika billing tidak dapat dibaca atau snapshot >24 jam, hentikan penambahan schedule/rerun nonpenting dan lanjutkan kerja lokal/read-only/penyimpanan tanpa Actions. Pada incident anggaran aktif, mode pause tetap berlaku sampai snapshot bulan baru diverifikasi; reset kalender saja tidak otomatis membuka gate.
- Sesudah run, rekam run/job ID, attempt, durasi, menit dibulatkan, hasil, estimasi-versus-aktual, dan sisa konservatif dengan timestamp dalam receipt privat terpisah, lalu selesaikan reservasinya. Jika belum selesai, tulis pending; jangan polling rapat. Gabungkan receipt pada handoff; pencatatan receipt tidak boleh memicu Actions baru.
- Lonjakan >2x estimasi atau >30 menit tak direncanakan per sesi: hentikan pemicu berulang, simpan bukti, beri tahu pemilik pada sesi berjalan. Jangan membuat scheduled monitor baru hanya untuk memantau biaya monitor lama.
- Gunakan filter path sebelum runner dialokasikan; pengecekan di langkah pertama tetap memakai menit. Jadikan maintenance satu kali manual, beri timeout/concurrency, hindari loop auto-repair/auto-commit. Jadwal yang sudah ada wajib dihitung sebelum diperluas.
- `AGENTS.md` adalah instruksi untuk agen, bukan hard limit GitHub. Jangan mengklaim seluruh akun terlindungi otomatis hanya karena dokumen ini ada. Bedakan policy, YAML yang sudah dibatasi, dan hard spending limit di billing.
- Repo baru harus menerima bagian ini sebelum workflow dibuat. Pertahankan satu versi policy dan catat penyebarannya; jangan membuat workflow berbayar untuk menyinkronkan policy. Jangan menyebarkan rincian billing/receipt privat ke repo publik.

<!-- END SHARED ACTIONS BUDGET POLICY v1 -->

<!-- BEGIN SOURCE SNAPSHOT REUSE POLICY v1 -->
## Baca snapshot sebelum mengambil sumber lagi

- Pertanyaan atau permintaan laporan bukan izin untuk menjalankan collector, dispatch Actions, atau mengambil ulang API. Cari snapshot JSON/manifest yang sudah ada pada repo proyek terkait terlebih dahulu.
- Gunakan nilai yang sudah dibaca dalam sesi ini selama sumbernya sama. Cantumkan waktu pengumpulan dan rentang data; jangan mengganti timestamp lama menjadi sekarang atau menyebut data lama sebagai live.
- Jika perlu data lebih rinci, baca shard/raw snapshot yang sama. Jangan mengambil ulang semua domain hanya karena satu bagian laporan belum dibaca.
- Pisahkan refresh sumber dari pemrosesan laporan. Builder laporan harus offline, tanpa kredensial dan tanpa fallback HTTP tersembunyi. Error/skema berubah menjaga snapshot terakhir; tidak memicu retry/fanout tanpa batas.
- Refresh baru hanya jika diminta eksplisit atau ada jadwal refresh yang memang disetujui dan masih dalam budget. TTL hanya mencegah pengambilan terlalu cepat; TTL kedaluwarsa tidak dengan sendirinya memberi izin fetch.
- Satu sumber/window/domain diambil sekali dan dibagikan ke semua laporan yang membutuhkannya. Simpan raw JSON yang diperlukan beserta generated_at, rentang, cap/sampling, dan identitas sumber; shard/kompres jika besar. Jangan membuat duplikat dataset di setiap repo tanpa kebutuhan.
- Commit hasil dan snapshot penting segera; jangan menahan satu-satunya salinan di artifact yang akan kedaluwarsa. Bedakan cache ringkas dari raw evidence; jangan mengaku bisa membangun ulang raw detail yang sebelumnya tidak pernah disimpan.
<!-- END SOURCE SNAPSHOT REUSE POLICY v1 -->

<!-- BEGIN OWNER ACTIONS VISIBILITY POLICY v2 -->
## Owner directive: GitHub Actions by repository visibility (2026-11-05)

This is the latest owner instruction. It overrides conflicting older shared Actions budget, reservation, notification, and pause rules **for public repositories only**.

- PUBLIC repositories: use standard GitHub-hosted Actions freely, as often as needed for CI/CD, tests, collectors, validation, monitoring, deployment, and refreshes. Do not reduce frequency, disable or migrate a working workflow solely to save included Actions minutes. No private-budget snapshot, master-ledger reservation, or advance quota notice is required for ordinary public-repo Actions.
- PRIVATE repositories: retain the existing shared Actions budget policy, cost monitoring, reservations, and safeguards. Do not interpret this public-repo exception as permission to spend private Actions minutes without checks.
- Verify actual repository visibility before applying either rule. Downstream private workflows, larger/paid runners, Cloudflare resources, and other billable services remain separate; do not enable paid overage without owner authorization.
- Preserve functional workflows. Validate replacements before disabling existing workflows, except when the owner explicitly requests otherwise. Prevent destructive data updates, concurrency conflicts, and runaway trigger loops.
- Reliability and comprehensive automation take priority over GitHub Actions minute conservation in PUBLIC repositories.
<!-- END OWNER ACTIONS VISIBILITY POLICY v2 -->
