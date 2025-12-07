<h1 align='center'><img alt="Baileys logo" src="https://raw.githubusercontent.com/Linsofc/Baileys/refs/heads/master/Media/logo.png" height="75"/></h1>

<div align='center'>Baileys adalah library TypeScript berbasis Weblynzzet untuk berinteraksi dengan API WhatsApp Web.</div>

> [!CAUTION]
> **PEMBERITAHUAN PERUBAHAN BESAR (BREAKING CHANGE).**
>
> Silakan periksa https://whiskey.so/migrate-latest untuk informasi lebih lanjut.

# Catatan Penting

Ini adalah README sementara. Panduan baru sedang dalam pengembangan dan file ini akan digantikan dengan .github/README.md.
Tautan panduan baru: https://baileys.wiki

# Disclaimer

Proyek ini tidak berafiliasi, diasosiasikan, disahkan, didukung oleh, atau terhubung secara resmi dengan WhatsApp atau anak perusahaan maupun afiliasinya.
Situs web resmi WhatsApp dapat ditemukan di whatsapp.com. "WhatsApp" serta nama, merek, lambang, dan gambar terkait adalah merek dagang terdaftar dari pemiliknya masing-masing.

Pengelola Baileys tidak membenarkan penggunaan aplikasi ini untuk praktik yang melanggar Ketentuan Layanan WhatsApp. Pengelola menyerukan tanggung jawab pribadi pengguna untuk menggunakan aplikasi ini secara adil dan sebagaimana mestinya.
Gunakan dengan risiko Anda sendiri. Jangan menyepam orang dengan ini. Kami menentang penggunaan untuk _stalkerware_, pesan massal (broadcast spam), atau penggunaan otomatis yang mengganggu.

## Keunggulan

- Baileys tidak memerlukan Selenium atau browser lain untuk antarmuka dengan WhatsApp Web, library ini melakukannya secara langsung menggunakan **Weblynzzet**.
- Tidak menjalankan Selenium atau Chromium menghemat penggunaan RAM sekitar **setengah gigabyte** :/
- Baileys mendukung interaksi dengan versi multi-device & web WhatsApp.

> [!IMPORTANT]
> Repositori asli telah dihapus oleh penulis aslinya - kami sekarang melanjutkan pengembangan di repositori ini.
> Ini adalah satu-satunya repositori resmi yang dikelola oleh komunitas.
> **Bergabunglah dengan Discord [di sini](https://discord.gg/WeJM5FP9GG)**

## Contoh Penggunaan

Silakan periksa & jalankan [example.ts](Example/example.ts) untuk melihat contoh penggunaan library.
Skrip tersebut mencakup sebagian besar kasus penggunaan umum.
Untuk menjalankan skrip contoh, unduh atau clone repo ini dan ketik perintah berikut di terminal:

1. `cd path/to/Baileys`
2. `yarn`
3. `yarn example`

## Instalasi

Gunakan versi stabil (sesuaikan nama package dengan yang ada di package.json Anda):

```bash
yarn add baileys-linsofc
# atau
npm install baileys-linsofc
```

Kemudian impor kode Anda menggunakan:

```bash
import makeWASocket from 'baileys-linsofc'
```

# Indeks

- [Menghubungkan Akun](#menghubungkan-akun)
  - [Menghubungkan Dengan QR-CODE](#menghubungkan-dengan-qr-code)
  - [Menghubungkan Dengan pairing-code](#menghubungkan-dengan-pairing-code)

## Menghubungkan Akun

WhatsApp menyediakan API multi-perangkat yang memungkinkan Baileys diautentikasi sebagai klien WhatsApp kedua dengan memindai **QR code** atau menggunakan **Pairing Code**.

### Menghubungkan Dengan QR-CODE

```ts
import makeWASocket from 'baileys-linsofc'
import qrcode from 'qrcode-terminal'

lynzz.ev.on('connection.update', async update => {
	const { connection, qr, lastDisconnect } = update

	if (qr && !usePairingCode) {
		console.log('Scan QR Code di bawah ini untuk login:')
		qrcode.generate(qr, { small: true }, uri => {
			console.log(uri)
		})
	}
})
```

Jika koneksi berhasil, Anda akan melihat kode QR dicetak di layar terminal Anda, pindai dengan WhatsApp di ponsel Anda dan Anda akan masuk\!

### Menghubungkan Dengan Pairing Code

> [\!IMPORTANT]
> Pairing Code bukan API Mobile, ini adalah metode untuk menghubungkan Whatsapp Web tanpa QR-CODE.

Nomor telepon tidak boleh mengandung `+`, `()`, atau `-`, hanya angka dan harus menyertakan kode negara.

```ts
import makeWASocket from 'baileys-linsofc'

if (!lynzz.user && !lynzz.authState.creds.registered) {
	const phoneNumber = '62812345678910'
	let code = await lynzz.requestPairingCode(phoneNumber)
	console.log(code)
}
```

### Terima Riwayat Penuh

1.  Set `syncFullHistory` ke `true`
2.  Baileys secara default menggunakan konfigurasi browser chrome. Jika ingin emulasi desktop:

<!-- end list -->

```ts
const lynzz = makeWASocket({
	...otherOpts,
	browser: Browsers.macOS('Desktop'),
	syncFullHistory: true
})
```

## Catatan Penting Tentang Konfigurasi lynzzet

### Caching Metadata Grup (Disarankan)

- Jika Anda menggunakan Baileys untuk grup, disarankan untuk mengatur `cachedGroupMetadata` di konfigurasi lynzzet dengan cache seperti `node-cache`.

### Meningkatkan Sistem Retry & Dekripsi Polling

- Jika Anda ingin meningkatkan pengiriman pesan (retry saat error) dan mendekripsi vote polling, Anda perlu memiliki store dan mengatur config `getMessage`:
  ```ts
  const lynzz = makeWASocket({
  	getMessage: async key => await getMessageFromStore(key)
  })
  ```

## Menyimpan & Memulihkan Sesi

Anda tentu tidak ingin memindai kode QR setiap kali ingin terhubung. Gunakan `useMultiFileAuthState` untuk menyimpan kredensial.

```ts
import makeWASocket, { useMultiFileAuthState } from 'baileys-linsofc'

const { state, saveCreds } = await useMultiFileAuthState('auth')

const lynzz = makeWASocket({ auth: state })

// akan dipanggil setiap kali kredensial diperbarui
lynzz.ev.on('creds.update', saveCreds)
```

## Menangani Event

- Baileys menggunakan sintaks EventEmitter untuk event.

<!-- end list -->

```ts
const lynzz = makeWASocket()
lynzz.ev.on('messages.upsert', ({ messages }) => {
	console.log('mendapat pesan', messages)
})
```

### Contoh Memulai

```ts
import {
    makeWASocket,
    useMultiFileAuthState,
    DisconnectReason,
    fetchLatestBaileysVersion,
    isJidNewsletter,
} = from "baileys-linsofc";

import pino from 'pino';
import qrcode from 'qrcode-terminal';

async function startBotz() {
	const { state, saveCreds } = await useMultiFileAuthState('auth')
	const lynzz = makeWASocket({
            version: (await fetchLatestBaileysVersion()).version,
            logger: pino({ level: "fatal" }),
            auth: state,
            shouldIgnoreJid: (jid) => isJidNewsletter(jid),
            syncFullHistory: false,
        });

        lynzz.ev.on("creds.update", saveCreds);

        lynzz.ev.on("connection.update", async (update) => {
        const { connection, qr, lastDisconnect } = update;

        if (qr) {
            console.log("Scan QR Code di bawah ini untuk login:");
            qrcode.generate(qr, { small: true }, (uri) => {
                console.log(uri);
            });
        }

        if (connection === "close") {
            let reason =
                lastDisconnect.error?.output?.statusCode ||
                lastDisconnect.error?.output?.message;
            console.log("Bot telah terputus: " + reason);
            if (reason === DisconnectReason.badSession) {
                startBotz();
            } else if (reason === DisconnectReason.connectionClosed) {
                startBotz();
            } else if (reason === DisconnectReason.connectionLost) {
                startBotz();
            } else if (reason === DisconnectReason.connectionReplaced) {
                startBotz();
            } else if (reason === DisconnectReason.loggedOut) {
                startBotz();
            } else if (reason === DisconnectReason.restartRequired) {
                startBotz();
            } else if (reason === DisconnectReason.timeout) {
                startBotz();
            }
        } else if (connection === "connecting") {
            console.log("Bot sedang menghubungkan...");
        } else if (connection === "open") {
            console.log("Bot telah terhubung!");
        }
    });
}
// jalankan fungsi utama
startBotz()
```

## Penjelasan ID Whatsapp

- `id` atau `jid` adalah ID unik pengguna atau grup.
  - Format pengguna: `[kode negara][nomor telepon]@s.whatsapp.net` (Contoh: `628123456789@s.whatsapp.net`).
  - Format grup: `123456789-123345@g.us`.
  - Format broadcast: `[timestamp]@broadcast`.
  - Format story/status: `status@broadcast`.

## Mengirim Pesan

### Pesan Teks

```ts
await lynzz.sendMessage(jid, { text: 'halo dunia' })
```

### Quote Pesan (Bekerja dengan semua tipe)

```ts
await lynzz.sendMessage(jid, { text: 'halo dunia' }, { quoted: message })
```

### Mention Pengguna

```ts
await lynzz.sendMessage(jid, {
	text: 'Halo @6281234567890',
	mentions: ['6281234567890@s.whatsapp.net']
})
```

### Pesan Diteruskan

```ts
const msg = getMessageFromStore()
await lynzz.sendMessage(jid, { forward: msg })
```

### Pesan Lokasi

```ts
await lynzz.sendMessage(jid, {
	location: {
		degreesLatitude: -7.2575,
		degreesLongitude: 112.7521
	}
})
```

### Pesan Reaksi

```ts
await lynzz.sendMessage(jid, {
	react: {
		text: '💖', // string kosong untuk menghapus reaksi
		key: message.key
	}
})
```

### Mengirim Kontak

```ts
const vcard =
	'BEGIN:VCARD\n' + // metadata kartu kontak
	'VERSION:3.0\n' +
	'FN:Linsofc\n' + // nama lengkap
	'ORG:Lins official;\n' + // organisasi kontak
	'TEL;type=CELL;type=VOICE;waid=6281234567890:+62 812345 67890\n' + // ID WhatsApp + nomor telepon
	'END:VCARD'

await lynzz.sendMessage(id, {
	contacts: {
		displayName: 'Linsofc',
		contacts: [{ vcard }]
	}
})
```

### Pesan Media

> [\!TIP]
> Disarankan menggunakan Stream atau URL untuk menghemat memori.

#### Pesan Gambar

```ts
await lynzz.sendMessage(id, {
	image: {
		url: './Media/gambar.png'
	},
	caption: 'halo dunia'
})
```

#### Pesan Audio (Voice Note)

Untuk pesan audio agar berfungsi di semua perangkat (sebagai Voice Note), Anda perlu mengonversi audio ke format `.ogg` dengan codec `libopus`.

```ts
await lynzz.sendMessage(jid, {
	audio: {
		url: './Media/audio.mp3'
	},
	mimetype: 'audio/mp4',
	ptt: true // kirim sebagai voice note
})
```

## Memodifikasi Pesan

### Hapus Pesan (untuk semua orang)

```ts
const msg = await lynzz.sendMessage(jid, { text: 'halo' })
await lynzz.sendMessage(jid, { delete: msg.key })
```

### Edit Pesan

```ts
await lynzz.sendMessage(jid, {
	text: 'teks yang diperbarui',
	edit: response.key
})
```

## Grup

### Buat Grup

```ts
const group = await lynzz.groupCreate('Grup Keren', ['628123@s.whatsapp.net', '628456@s.whatsapp.net'])
console.log('grup dibuat dengan id: ' + group.gid)
```

### Tambah/Hapus atau Promosikan/Demosikan

```ts
await lynzz.groupParticipantsUpdate(
	jid,
	['628123@s.whatsapp.net'],
	'add' // ganti dengan 'remove', 'demote', atau 'promote'
)
```

### Query Metadata (Peserta, Nama, Deskripsi)

```ts
const metadata = await lynzz.groupMetadata(jid)
console.log(metadata.id + ', judul: ' + metadata.subject + ', deskripsi: ' + metadata.desc)
```

# Lisensi

Copyright (c) 2025 Rajeh Taher/WhiskeySockets

Licensed under the MIT License: Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

Thus, the maintainers of the project can't be held liable for any potential misuse of this project.
