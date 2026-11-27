# full-stack-builder

Skill untuk Claude Code yang memandu pembuatan aplikasi dari ide sampai deploy, tahap demi tahap. Cocok untuk pemula yang bingung mulai dari mana: skill ini mewawancarai kamu dulu (mau bikin apa, buat siapa, apa yang dibutuhkan), lalu menyusun rencana, desain, tiket kerja, kode, tes, review, dan deploy.

## Isi

- `skills/full-stack-builder/SKILL.md` : definisi skill.

## 13 tahap

| Tahap | Fungsi | Model | Effort |
|---|---|---|---|
| 1 Requirements | Wawancara kebutuhan, glosarium, ADR | model sesi | high |
| 2 Spec | Spesifikasi dengan kriteria penerimaan yang bisa dites | opus | high |
| 3 UI/UX | Layar, alur, token desain | sonnet | medium |
| 4 Architecture | Stack, model data, kontrak API, ADR | opus | xhigh |
| 5 Tickets | Potong jadi tugas kecil berurutan | haiku | low |
| 6 Implementation | Tes dulu, kode minimum, satu tiket satu commit | sonnet (haiku untuk tiket kecil, opus untuk auth/uang/data) | medium (high untuk auth/data) |
| 7 E2E | Tes alur kritis dengan Playwright | sonnet | medium |
| 8-11 Review | Keamanan, kualitas, code review, verifikasi bukti | opus | high |
| 12 Deploy | Daftar cek pra-deploy, deploy, smoke test | model sesi | medium |
| 13 Docs | README, ADR, serah terima | haiku | low |

Ada 6 gerbang persetujuan (A sampai F). Skill berhenti dan meminta persetujuan eksplisit kamu sebelum lanjut: requirements, spec, desain, arsitektur, tiket, dan deploy. Tabel routing yang berlaku ada di `SKILL.md`.

## Pasang

Salin folder `skills/full-stack-builder` ke folder skill Claude Code kamu.

```powershell
git clone https://github.com/sufi-0119/full-stack-builder.git
Copy-Item -Recurse full-stack-builder\skills\full-stack-builder $env:USERPROFILE\.claude\skills\
```

Lalu di Claude Code ketik `/full-stack-builder`.

## Ketergantungan

Skill ini adalah orkestrator. Tiap tahap memuat skill lain yang harus sudah terpasang di mesin kamu, yaitu: `grill-with-docs`, `to-spec`, `frontend-design`, `ui-ux-pro-max`, `domain-modeling`, `codebase-design`, `to-tickets`, `implement`, `tdd`, `systematic-debugging`, `playwright-best-practices`, `security-audit`, `web-design-guidelines`, `code-review`, `verification-before-completion`, `threat-model`, `deploy-to-vercel`, `handoff`, dan `setup-matt-pocock-skills`. Repo ini tidak menyertakan skill-skill tersebut.

Nama model di tabel (`opus`, `sonnet`, `haiku`) dan aturan jawab dalam bahasa Indonesia ditulis untuk setup penulis. Sesuaikan di `SKILL.md` bila perlu.

## Lisensi

MIT. Lihat `LICENSE`.
