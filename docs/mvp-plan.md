# E-Budget — MVP Plan

## Tujuan

Aplikasi e-budgeting internal berbasis **Power Apps Code Apps** (React +
TypeScript) di atas **Dataverse**, menggantikan proses approval budget
manual (CEP & CER) dengan alur digital yang auditable. Lihat
[`architecture.md`](./architecture.md) untuk keputusan teknis.

## Aktor / Role

| Role                   | Deskripsi                                                                                                                       |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **Department Manager** | Mengajukan CEP untuk departemennya, mengajukan CER terhadap CEP yang sudah approved, meng-edit & resubmit saat status `Revise`. |
| **CEP Approver**       | Approve / reject / minta revisi atas CEP yang masuk.                                                                            |
| **CER Approver**       | Approve / reject / minta revisi atas CER yang masuk (bisa role terpisah dari CEP Approver, misalnya Finance).                   |

Role dipetakan lewat Dataverse security role, bukan tabel role custom —
mengikuti mekanisme akses yang sudah ada di Power Platform (KISS: tidak
membangun sistem permission sendiri).

## Entity Dataverse (sketsa)

Belum dibuat di Dataverse — ini sketsa awal untuk memandu `pac code
add-data-source`, akan disesuaikan saat schema benar-benar dibuat.

### `cep` — Capital Expenditure Plan

| Field                         | Tipe            | Catatan                                                |
| ----------------------------- | --------------- | ------------------------------------------------------ |
| `cep_id`                      | primary key     |                                                        |
| `title`                       | text            |                                                        |
| `department`                  | lookup / choice |                                                        |
| `fiscal_year`                 | choice / number |                                                        |
| `category`                    | choice          | kategori capex                                         |
| `amount_planned`              | currency        |                                                        |
| `justification`               | multiline text  |                                                        |
| `status`                      | choice          | `Draft \| Submitted \| Approved \| Rejected \| Revise` |
| `submitted_by`                | lookup (user)   |                                                        |
| `approver`                    | lookup (user)   |                                                        |
| `approver_comment`            | multiline text  | diisi saat reject/revise                               |
| `submitted_on` / `decided_on` | datetime        |                                                        |

### `cer` — Capital Expenditure Request

| Field                         | Tipe           | Catatan                                                        |
| ----------------------------- | -------------- | -------------------------------------------------------------- |
| `cer_id`                      | primary key    |                                                                |
| `cep`                         | lookup → `cep` | wajib merujuk CEP berstatus `Approved`                         |
| `amount_requested`            | currency       | tidak boleh melebihi sisa `amount_planned` milik `cep` terkait |
| `purpose`                     | multiline text | keperluan pencairan                                            |
| `status`                      | choice         | `Draft \| Submitted \| Approved \| Rejected \| Revise`         |
| `submitted_by`                | lookup (user)  |                                                                |
| `approver`                    | lookup (user)  |                                                                |
| `approver_comment`            | multiline text |                                                                |
| `submitted_on` / `decided_on` | datetime       |                                                                |

Riwayat approve/reject/revise cukup disimpan sebagai kolom
`approver_comment` + `decided_on` pada entity itu sendiri untuk MVP —
**bukan** tabel audit-log terpisah (lihat Non-Goals).

## Alur Utama

### 1. Pengajuan & approval CEP

1. Manager membuat CEP baru → status `Draft`.
2. Manager submit → status `Submitted`.
3. CEP Approver membuka daftar CEP `Submitted`, memilih salah satu:
   - **Approve** → status `Approved`.
   - **Reject** → status `Rejected` (selesai, tidak bisa diedit lagi).
   - **Revise** → status `Revise` + `approver_comment` wajib diisi.
4. Jika `Revise`, Manager edit CEP lalu submit ulang → kembali ke `Submitted`.

### 2. Pencairan & approval CER

1. Manager membuat CER baru, hanya bisa memilih `cep` berstatus `Approved`
   dan dengan sisa budget > 0.
2. Manager submit → status `Submitted`.
3. CER Approver membuka daftar CER `Submitted`, memilih:
   - **Approve** → status `Approved` (dana dianggap cair).
   - **Reject** → status `Rejected`.
   - **Revise** → status `Revise` + `approver_comment` wajib diisi.
4. Jika `Revise`, Manager edit CER lalu submit ulang.

CEP dan CER memakai state machine status yang identik — didokumentasikan di
[`architecture.md`](./architecture.md#cep--cer-status-workflow) sebagai
kandidat ekstraksi shared logic setelah keduanya benar-benar dibangun.

## Non-Goals (di luar scope MVP)

- Multi-level / berjenjang approval (misal butuh 2 approver berurutan).
- Notifikasi email/Teams saat status berubah.
- Dashboard/reporting/rekap anggaran.
- Lampiran file (invoice, dokumen pendukung).
- Multi-currency.
- Delegasi approval (approver pengganti saat cuti).
- Tabel audit-log terpisah — histori cukup dari kolom status + comment +
  timestamp di entity yang sama.

Semua ini valid untuk fase berikutnya, tapi sengaja tidak masuk MVP supaya
alur inti (submit → approve/reject/revise) bisa selesai & diverifikasi
lebih dulu.

## Milestone

1. **Foundation** _(selesai di task ini)_ — boilerplate React/TS, struktur
   folder, docs, `CLAUDE.md`, skill `/investigate-e-budget` &
   `/develop-e-budget`.
2. **Dataverse wiring** — `pac code init`, buat tabel `cep`/`cer` di
   Dataverse, `pac code add-data-source`, isi `src/models` & `src/services`.
3. **CEP flow** — form submit CEP (Manager) + daftar approval CEP
   (Approver), TDD dari service layer sampai komponen.
4. **CER flow** — sama seperti CEP, plus validasi sisa budget terhadap CEP
   terkait.
5. **Polish** — role/permission review, error states, deploy ke environment
   lewat `pac code push`.

Setiap milestone diverifikasi lewat `npm run test`, `npm run lint`,
`npm run build` sebelum lanjut ke milestone berikutnya.
