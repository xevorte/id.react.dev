---
title: useEffect
---

<Intro>

`useEffect` adalah React Hook yang memungkinkan Anda [menyinkronkan komponen dengan sistem eksternal.](/learn/synchronizing-with-effects)

```js
useEffect(setup, dependencies?)
```

</Intro>

<InlineToc />

---

## Referensi {/*reference*/}

### `useEffect(setup, dependencies?)` {/*useeffect*/}

Panggil `useEffect` di tingkat atas komponen Anda untuk mendeklarasikan sebuah Effect:

```js
import { useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [serverUrl, roomId]);
  // ...
}
```

[Lihat contoh-contoh lainnya di bawah ini.](#usage)

#### Parameters {/*parameters*/}

* `setup`: Fungsi dengan logika Effect Anda. Fungsi setup Anda juga dapat secara opsional mengembalikan fungsi *cleanup*. Ketika komponen Anda pertama kali ditambahkan ke DOM, React akan menjalankan fungsi setup Anda. Setelah setiap render ulang dengan dependensi yang berubah, React akan menjalankan fungsi cleanup terlebih dahulu (jika Anda menyediakannya) dengan nilai-nilai yang lama, lalu jalankan fungsi setup Anda dengan nilai yang baru. Setelah komponen Anda dihapus dari DOM, React akan menjalankan fungsi cleanup untuk terakhir kalinya.

* **opsional** `dependensi`: Daftar semua nilai-nilai reaktif yang direferensikan di dalam kode `setup`. Nilai-nilai reaktif meliputi props, state, dan semua variabel dan fungsi yang dideklarasikan secara langsung di dalam badan komponen Anda. Jika linter Anda adalah [dikonfigurasi untuk React](/learn/editor-setup#linting), ini akan memverifikasi bahwa setiap nilai reaktif ditentukan dengan benar sebagai dependensi. Daftar dependensi harus memiliki jumlah item yang konstan dan ditulis sebaris seperti `[dep1, dep2, dep3]`. React akan membandingkan setiap dependensi dengan nilai sebelumnya menggunakan fungsi perbandingan [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is). Jika Anda menghilangkan argumen ini, Effect Anda akan dijalankan ulang setelah setiap render ulang komponen. [Lihat perbedaan antara melewatkan senarai dependensi, senarai kosong, dan tidak ada dependensi sama sekali.](#examples-dependencies)

#### Returns {/*returns*/}

`useEffect` mengembalikan `undefined`.

#### Perhatian {/*caveats*/}

* `useEffect` adalah sebuah Hook, jadi Anda hanya bisa memanggilnya **di tingkat atas komponen Anda** atau Hooka Anda sendiri. Anda tidak dapat memanggilnya di dalam loop atau kondisi. Jika Anda membutuhkannya, ekstrak sebuah komponen baru dan pindahkan state ke dalamnya.

* Jika Anda **tidak sedang mencoba menyinkronkan dengan beberapa sistem eksternal,** [Anda mungkin tidak memerlukan Effect.](/learn/you-might-not-need-an-effect)

* Ketika Strict Mode aktif, React akan **menjalankan satu siklus development-only setup+cleanup** sebelum setup pertama yang sebenarnya. Ini adalah stress-test yang memastikan bahwa logika cleanup Anda "mencerminkan" logika setup Anda dan menghentikan atau membatalkan apa pun yang dilakukan setup. Jika hal ini menyebabkan masalah, [mengimplementasikan fungsi cleanup.](/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development)

* Jika beberapa dependensi Anda adalah objek atau fungsi yang didefinisikan di dalam komponen, ada risiko bahwa mereka akan **menyebabkan Effect berjalan kembali lebih sering daripada yang dibutuhkan.** Untuk mengatasinya, hapus [objek](#removing-unnecessary-object-dependencies) yang tidak perlu dan [fungsi](#removing-unnecessary-function-dependencies) dependensi. Anda juga dapat [mengekstrak pembaruan state](#updating-state-based-on-previous-state-from-an-effect) dan melakukan [logika non-reaktif](#reading-the-latest-props-and-state-from-an-effect) di luar Effect Anda.

* Jika Effect Anda tidak disebabkan oleh interaksi (seperti mengeklik), React akan membiarkan browser **menampilkan layar yang telah diperbarui terlebih dahulu sebelum menjalankan Effect Anda.** jika Effect Anda melakukan sesuatu yang bersifat visual (misalnya, memposisikan sebuah tooltip), dan penundaannya terlihat jelas (misalnya, berkedip-kedip), ganti `useEffect` dengan [`useLayoutEffect`.](/reference/react/useLayoutEffect)

* Meskipun Effect Anda disebabkan oleh interaksi (seperti mengeklik), **browser dapat menampilkan ulang layar sebelum memproses pembaruan status di dalam Effect Anda.** Biasanya, itulah yang Anda inginkan. Namun, jika Anda harus memblokir browser agar tidak menampilkan ulang layar, Anda perlu mengganti `useEffect` dengan [`useLayoutEffect`.](/reference/react/useLayoutEffect)

* Effects **hanya akan berjalan pada sisi client** Mereka tidak berjalan selama rendering server.

---

## Penggunaan {/*usage*/}

### Menghubungkan ke sistem eksternal {/*connecting-to-an-external-system*/}

Beberapa komponen harus tetap terhubung ke jaringan, beberapa API browser, atau library pihak ketiga, ketika komponen-komponen tersebut ditampilkan pada laman. Sistem-sistem ini tidak dikontrol oleh React, sehingga mereka disebut *eksternal.*

Untuk [menghubungkan komponen Anda ke beberapa sistem eksternal,](/learn/synchronizing-with-effects) panggil `useEffect` di tingkat atas komponen Anda:

```js [[1, 8, "const connection = createConnection(serverUrl, roomId);"], [1, 9, "connection.connect();"], [2, 11, "connection.disconnect();"], [3, 13, "[serverUrl, roomId]"]]
import { useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
  	const connection = createConnection(serverUrl, roomId);
    connection.connect();
  	return () => {
      connection.disconnect();
  	};
  }, [serverUrl, roomId]);
  // ...
}
```

Anda perlu memberikan dua argumen ke `useEffect`:

1. Sebuah *fungsi setup* dengan <CodeStep step={1}>kode setup</CodeStep> yang terhubung ke sistem tersebut.
   - Ini harus mengembalikan sebuah *fungsi cleanup* dengan <CodeStep step={2}>kode cleanup</CodeStep> yang terputus dari sistem tersebut.
2. Sebuah <CodeStep step={3}>daftar dependensi</CodeStep> termasuk setiap nilai dari komponen Anda yang digunakan di dalam fungsi-fungsi tersebut.

**React memanggil fungsi setup dan cleanup kapan pun diperlukan, yang mungkin terjadi beberapa kali:**

1. <CodeStep step={1}>Kode setup</CodeStep> Anda berjalan ketika komponen Anda ditambahkan ke laman *(mounts)*.
2. Setelah setiap render ulang komponen Anda di mana <CodeStep step={3}>dependensi</CodeStep> telah berubah:
   - Pertama, <CodeStep step={2}>kode cleanup</CodeStep> Anda berjalan dengan props dan state yang sudah lama.
   - Kemudian, <CodeStep step={1}>kode setup</CodeStep> Anda berjalan dengan props dan state yang baru.
3. <CodeStep step={2}>Kode cleanup</CodeStep> Anda berjalan untuk terakhir kalinya setelah komponen Anda dihapus dari laman *(unmounts).*

**Mari kita ilustrasikan urutan ini untuk contoh di atas.**  

Ketika komponen `ChatRoom` di atas ditambahkan ke laman, komponen ini akan terhubung ke ruang obrolan dengan `serverUrl` dan `roomId` awal. Jika salah satu dari `serverUrl` atau `roomId` berubah sebagai hasil dari render ulang (misalnya, jika pengguna memilih ruang obrolan yang berbeda dalam dropdown), Effect Anda akan *memutuskan sambungan dari ruang sebelumnya, dan menyambungkannya ke ruang berikutnya.* Ketika komponen `ChatRoom` dihapus dari laman, Effect Anda akan terputus untuk terakhir kalinya.

**Untuk [membantu Anda menemukan bug,](/learn/synchronizing-with-effects#step-3-add-cleanup-if-needed) di dalam development React menjalankan <CodeStep step={1}>setup</CodeStep> dan <CodeStep step={2}>cleanup</CodeStep> ekstra satu kali sebelum <CodeStep step={1}>setup</CodeStep>.** Ini adalah stress-test yang memverifikasi logika Effect Anda apakah sudah diimplementasikan dengan benar. Jika hal ini menyebabkan masalah yang terlihat, fungsi cleanup Anda kehilangan beberapa logika. Fungsi cleanup harus menghentikan atau membatalkan apa pun yang dilakukan oleh fungsi setup. Aturan praktisnya adalah bahwa pengguna seharusnya tidak dapat membedakan antara setup yang dipanggil sekali (seperti dalam produksi) dan sebuah *setup* → *cleanup* → *setup* urutan (seperti dalam development). [Lihat solusi umum].(/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development)

**Coba untuk [menulis setiap Effect sebagai proses independen](/learn/lifecycle-of-reactive-effects#each-effect-represents-a-separate-synchronization-process) dan [pikirkan tentang satu siklus setup/cleanup dalam satu waktu.](/learn/lifecycle-of-reactive-effects#thinking-from-the-effects-perspective)** Seharusnya tidak menjadi masalah apakah komponen Anda dipasang, diperbarui, atau dilepas. Saat logika cleanup Anda dengan benar "mencerminkan" logika setup, Effect Anda akan tangguh dalam menjalankan setup dan cleanup sesering yang diperlukan.

<Note>

Sebuah Effect memungkinkan Anda [menjaga komponen Anda tetap tersinkronisasi](/learn/synchronizing-with-effects) dengan beberapa sistem eksternal (seperti layanan chatting). Di sini, *sistem eksternal* berarti setiap bagian dari kode yang tidak dikontrol oleh React, seperti:

* Sebuah pengatur waktu yang dikelola dengan <CodeStep step={1}>[`setInterval()`](https://developer.mozilla.org/en-US/docs/Web/API/setInterval)</CodeStep> dan <CodeStep step={2}>[`clearInterval()`](https://developer.mozilla.org/en-US/docs/Web/API/clearInterval)</CodeStep>.
* Sebuah event subscription menggunakan <CodeStep step={1}>[`window.addEventListener()`](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener)</CodeStep> dan <CodeStep step={2}>[`window.removeEventListener()`](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/removeEventListener)</CodeStep>.
* Sebuah library animasi pihak ketiga dengan API seperti <CodeStep step={1}>`animation.start()`</CodeStep> dan <CodeStep step={2}>`animation.reset()`</CodeStep>.

**Jika Anda tidak menghubungkan ke sistem eksternal apa pun, [Anda mungkin tidak memerlukan Effect.](/learn/you-might-not-need-an-effect)**

</Note>

<Recipes titleText="Contoh menghubungkan ke sistem eksternal" titleId="examples-connecting">

#### Menghubungkan ke server obrolan {/*connecting-to-a-chat-server*/}

Dalam contoh ini, komponen `ChatRoom` menggunakan Effect untuk tetap terhubung ke sistem eksternal yang ditentukan dalam `chat.js`. Tekan "Open chat" untuk membuat komponen `ChatRoom` muncul. Sandbox ini berjalan dalam mode pengembangan, jadi ada siklus tambahan untuk menyambung dan memutuskan sambungan, seperti yang [dijelaskan di sini.](/learn/synchronizing-with-effects#step-3-add-cleanup-if-needed) Coba ubah `roomId` dan `serverUrl` menggunakan dropdown dan masukan, dan lihat bagaimana Effect terhubung kembali ke obrolan. Tekan "Close chat" untuk melihat Effect terputus untuk terakhir kalinya.

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId, serverUrl]);

  return (
    <>
      <label>
        Server URL:{' '}
        <input
          value={serverUrl}
          onChange={e => setServerUrl(e.target.value)}
        />
      </label>
      <h1>Welcome to the {roomId} room!</h1>
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [show, setShow] = useState(false);
  return (
    <>
      <label>
        Choose the chat room:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <button onClick={() => setShow(!show)}>
        {show ? 'Close chat' : 'Open chat'}
      </button>
      {show && <hr />}
      {show && <ChatRoom roomId={roomId} />}
    </>
  );
}
```

```js chat.js
export function createConnection(serverUrl, roomId) {
  // A real implementation would actually connect to the server
  return {
    connect() {
      console.log('✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

<Solution />

#### Memahami sebuah event browser global {/*listening-to-a-global-browser-event*/}

Dalam contoh ini, sistem eksternal adalah DOM browser itu sendiri. Biasanya, Anda akan menentukan event listeners khusus dengan JSX, tetapi Anda tidak dapat menyambungkannya dengan objek global [`window`](https://developer.mozilla.org/en-US/docs/Web/API/Window) dengan cara ini. Sebuah Effect memungkinkan Anda terhubung ke objek `window` dan listen ke events tersebut. Listening ke event `pointermove` memungkinkan Anda melacak posisi kursor (atau jari) dan memperbarui titik merah untuk bergerak bersamanya.

<Sandpack>

```js
import { useState, useEffect } from 'react';

export default function App() {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    function handleMove(e) {
      setPosition({ x: e.clientX, y: e.clientY });
    }
    window.addEventListener('pointermove', handleMove);
    return () => {
      window.removeEventListener('pointermove', handleMove);
    };
  }, []);

  return (
    <div style={{
      position: 'absolute',
      backgroundColor: 'pink',
      borderRadius: '50%',
      opacity: 0.6,
      transform: `translate(${position.x}px, ${position.y}px)`,
      pointerEvents: 'none',
      left: -20,
      top: -20,
      width: 40,
      height: 40,
    }} />
  );
}
```

```css
body {
  min-height: 300px;
}
```

</Sandpack>

<Solution />

#### Memicu sebuah animasi {/*triggering-an-animation*/}

Dalam contoh ini, sistem eksternal adalah library animasi di `animation.js`. Ini menyediakan kelas JavaScript yang disebut `FadeInAnimation` yang mengambil node DOM sebagai argumen dan mengekspos method `start()` dan `stop()` untuk mengontrol animasi. Komponen ini [menggunakan sebuah ref](/learn/manipulating-the-dom-with-refs) untuk mengakses node DOM yang mendasarinya. Effect membaca node DOM dari ref dan secara otomatis memulai animasi untuk node tersebut saat komponen muncul.

<Sandpack>

```js
import { useState, useEffect, useRef } from 'react';
import { FadeInAnimation } from './animation.js';

function Welcome() {
  const ref = useRef(null);

  useEffect(() => {
    const animation = new FadeInAnimation(ref.current);
    animation.start(1000);
    return () => {
      animation.stop();
    };
  }, []);

  return (
    <h1
      ref={ref}
      style={{
        opacity: 0,
        color: 'white',
        padding: 50,
        textAlign: 'center',
        fontSize: 50,
        backgroundImage: 'radial-gradient(circle, rgba(63,94,251,1) 0%, rgba(252,70,107,1) 100%)'
      }}
    >
      Welcome
    </h1>
  );
}

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(!show)}>
        {show ? 'Remove' : 'Show'}
      </button>
      <hr />
      {show && <Welcome />}
    </>
  );
}
```

```js animation.js
export class FadeInAnimation {
  constructor(node) {
    this.node = node;
  }
  start(duration) {
    this.duration = duration;
    if (this.duration === 0) {
      // Jump to end immediately
      this.onProgress(1);
    } else {
      this.onProgress(0);
      // Start animating
      this.startTime = performance.now();
      this.frameId = requestAnimationFrame(() => this.onFrame());
    }
  }
  onFrame() {
    const timePassed = performance.now() - this.startTime;
    const progress = Math.min(timePassed / this.duration, 1);
    this.onProgress(progress);
    if (progress < 1) {
      // We still have more frames to paint
      this.frameId = requestAnimationFrame(() => this.onFrame());
    }
  }
  onProgress(progress) {
    this.node.style.opacity = progress;
  }
  stop() {
    cancelAnimationFrame(this.frameId);
    this.startTime = null;
    this.frameId = null;
    this.duration = 0;
  }
}
```

```css
label, button { display: block; margin-bottom: 20px; }
html, body { min-height: 300px; }
```

</Sandpack>

<Solution />

#### Mengontrol sebuah dialog modal {/*controlling-a-modal-dialog*/}

Dalam contoh ini, sistem eksternal adalah DOM browser. Komponen `ModalDialog` merender sebuah elemen [`<dialog>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/dialog). Ini menggunakan Effect untuk menyinkronkan prop `isOpen` ke pemanggilan method [`showModal()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement/showModal) dan [`close()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement/close).

<Sandpack>

```js
import { useState } from 'react';
import ModalDialog from './ModalDialog.js';

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(true)}>
        Open dialog
      </button>
      <ModalDialog isOpen={show}>
        Hello there!
        <br />
        <button onClick={() => {
          setShow(false);
        }}>Close</button>
      </ModalDialog>
    </>
  );
}
```

```js ModalDialog.js active
import { useEffect, useRef } from 'react';

export default function ModalDialog({ isOpen, children }) {
  const ref = useRef();

  useEffect(() => {
    if (!isOpen) {
      return;
    }
    const dialog = ref.current;
    dialog.showModal();
    return () => {
      dialog.close();
    };
  }, [isOpen]);

  return <dialog ref={ref}>{children}</dialog>;
}
```

```css
body {
  min-height: 300px;
}
```

</Sandpack>

<Solution />

#### Melacak visibilitas elemen {/*tracking-element-visibility*/}

Dalam contoh ini, sistem eksternal sekali lagi adalah DOM browser. Komponen `App` menampilkan daftar panjang, lalu komponen `Box`, dan kemudian daftar panjang lainnya. Gulir daftar ke bawah. Perhatikan bahwa ketika komponen `Box` muncul di viewport, warna latar belakang berubah menjadi hitam. Untuk mengimplementasikan hal ini, komponen `Box` menggunakan Effect untuk mengelola sebuah [`IntersectionObserver`](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API). API browser ini memberi tahu Anda ketika elemen DOM terlihat di viewport.

<Sandpack>

```js
import Box from './Box.js';

export default function App() {
  return (
    <>
      <LongSection />
      <Box />
      <LongSection />
      <Box />
      <LongSection />
    </>
  );
}

function LongSection() {
  const items = [];
  for (let i = 0; i < 50; i++) {
    items.push(<li key={i}>Item #{i} (keep scrolling)</li>);
  }
  return <ul>{items}</ul>
}
```

```js Box.js active
import { useRef, useEffect } from 'react';

export default function Box() {
  const ref = useRef(null);

  useEffect(() => {
    const div = ref.current;
    const observer = new IntersectionObserver(entries => {
      const entry = entries[0];
      if (entry.isIntersecting) {
        document.body.style.backgroundColor = 'black';
        document.body.style.color = 'white';
      } else {
        document.body.style.backgroundColor = 'white';
        document.body.style.color = 'black';
      }
    });
    observer.observe(div, {
      threshold: 1.0
    });
    return () => {
      observer.disconnect();
    }
  }, []);

  return (
    <div ref={ref} style={{
      margin: 20,
      height: 100,
      width: 100,
      border: '2px solid black',
      backgroundColor: 'blue'
    }} />
  );
}
```

</Sandpack>

<Solution />

</Recipes>

---

### Membungkus banyak Effect di dalam Hook kustom {/*wrapping-effects-in-custom-hooks*/}

Effect adalah sebuah ["pintu darurat":](/learn/escape-hatches) Anda menggunakannya ketika Anda perlu "melangkah keluar dari React" dan ketika tidak ada solusi bawaan yang lebih baik untuk kasus penggunaan Anda. Jika Anda merasa sering perlu menulis banyak Effect secara manual, biasanya itu adalah tanda bahwa Anda perlu mengekstrak beberapa [Hook kustom](/learn/reusing-logic-with-custom-hooks) untuk perilaku umum yang diandalkan oleh komponen Anda.

Sebagai contoh, Hook kustom `useChatRoom` ini "menyembunyikan" logika Effect Anda di balik API yang lebih deklaratif:

```js {1,11}
function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}
```

Kemudian, Anda dapat menggunakannya dari komponen apa pun seperti ini:

```js {4-7}
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl
  });
  // ...
````

Ada juga banyak Hook kustom yang sangat baik untuk setiap tujuan yang tersedia di ekosistem React.

[Pelajari lebih lanjut tentang membungkus banyak Effect dalam Hooks kustom.](/learn/reusing-logic-with-custom-hooks)

<Recipes titleText="Contoh melakukan pembungkusan banyak Effect dalam Hook kustom" titleId="examples-custom-hooks">

#### Hook kustom `useChatRoom` {/*custom-usechatroom-hook*/}

Contoh ini identik dengan salah satu dari [contoh sebelumnya,](#examples-connecting) tetapi logikanya diekstrak ke Hook kustom.

<Sandpack>

```js
import { useState } from 'react';
import { useChatRoom } from './useChatRoom.js';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl
  });

  return (
    <>
      <label>
        Server URL:{' '}
        <input
          value={serverUrl}
          onChange={e => setServerUrl(e.target.value)}
        />
      </label>
      <h1>Welcome to the {roomId} room!</h1>
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [show, setShow] = useState(false);
  return (
    <>
      <label>
        Choose the chat room:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <button onClick={() => setShow(!show)}>
        {show ? 'Close chat' : 'Open chat'}
      </button>
      {show && <hr />}
      {show && <ChatRoom roomId={roomId} />}
    </>
  );
}
```

```js useChatRoom.js
import { useEffect } from 'react';
import { createConnection } from './chat.js';

export function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId, serverUrl]);
}
```

```js chat.js
export function createConnection(serverUrl, roomId) {
  // A real implementation would actually connect to the server
  return {
    connect() {
      console.log('✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

<Solution />

#### Hook kustom `useWindowListener` {/*custom-usewindowlistener-hook*/}

Contoh ini identik dengan salah satu dari [contoh sebelumnya,](#examples-connecting) tetapi logikanya diekstrak ke Hook kustom.

<Sandpack>

```js
import { useState } from 'react';
import { useWindowListener } from './useWindowListener.js';

export default function App() {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useWindowListener('pointermove', (e) => {
    setPosition({ x: e.clientX, y: e.clientY });
  });

  return (
    <div style={{
      position: 'absolute',
      backgroundColor: 'pink',
      borderRadius: '50%',
      opacity: 0.6,
      transform: `translate(${position.x}px, ${position.y}px)`,
      pointerEvents: 'none',
      left: -20,
      top: -20,
      width: 40,
      height: 40,
    }} />
  );
}
```

```js useWindowListener.js
import { useState, useEffect } from 'react';

export function useWindowListener(eventType, listener) {
  useEffect(() => {
    window.addEventListener(eventType, listener);
    return () => {
      window.removeEventListener(eventType, listener);
    };
  }, [eventType, listener]);
}
```

```css
body {
  min-height: 300px;
}
```

</Sandpack>

<Solution />

#### Hook kustom `useIntersectionObserver` {/*custom-useintersectionobserver-hook*/}

Contoh ini identik dengan salah satu dari [contoh sebelumnya,](#examples-connecting) tetapi logikanya diekstrak ke Hook kustom.

<Sandpack>

```js
import Box from './Box.js';

export default function App() {
  return (
    <>
      <LongSection />
      <Box />
      <LongSection />
      <Box />
      <LongSection />
    </>
  );
}

function LongSection() {
  const items = [];
  for (let i = 0; i < 50; i++) {
    items.push(<li key={i}>Item #{i} (keep scrolling)</li>);
  }
  return <ul>{items}</ul>
}
```

```js Box.js active
import { useRef, useEffect } from 'react';
import { useIntersectionObserver } from './useIntersectionObserver.js';

export default function Box() {
  const ref = useRef(null);
  const isIntersecting = useIntersectionObserver(ref);

  useEffect(() => {
   if (isIntersecting) {
      document.body.style.backgroundColor = 'black';
      document.body.style.color = 'white';
    } else {
      document.body.style.backgroundColor = 'white';
      document.body.style.color = 'black';
    }
  }, [isIntersecting]);

  return (
    <div ref={ref} style={{
      margin: 20,
      height: 100,
      width: 100,
      border: '2px solid black',
      backgroundColor: 'blue'
    }} />
  );
}
```

```js useIntersectionObserver.js
import { useState, useEffect } from 'react';

export function useIntersectionObserver(ref) {
  const [isIntersecting, setIsIntersecting] = useState(false);

  useEffect(() => {
    const div = ref.current;
    const observer = new IntersectionObserver(entries => {
      const entry = entries[0];
      setIsIntersecting(entry.isIntersecting);
    });
    observer.observe(div, {
      threshold: 1.0
    });
    return () => {
      observer.disconnect();
    }
  }, [ref]);

  return isIntersecting;
}
```

</Sandpack>

<Solution />

</Recipes>

---

### Mengontrol sebuah widget non-React {/*controlling-a-non-react-widget*/}

Terkadang, Anda ingin menjaga sistem eksternal tetap tersinkronisasi ke beberapa prop atau state milik komponen Anda.

Sebagai contoh, jika Anda memiliki widget peta pihak ketiga atau komponen pemutar video yang ditulis tanpa React, Anda dapat menggunakan sebuah Effect untuk memanggil method di dalamnya yang membuat state-nya sama dengan state komponen React Anda saat ini. Effect ini membuat instance dari kelas `MapWidget` yang didefinisikan di `map-widget.js`. Saat Anda mengubah prop `zoomLevel` pada komponen `Map`, Effect memanggil `setZoom()` pada instance kelas agar tetap tersinkronisasi:

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "leaflet": "1.9.1",
    "react": "latest",
    "react-dom": "latest",
    "react-scripts": "latest",
    "remarkable": "2.0.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

```js App.js
import { useState } from 'react';
import Map from './Map.js';

export default function App() {
  const [zoomLevel, setZoomLevel] = useState(0);
  return (
    <>
      Zoom level: {zoomLevel}x
      <button onClick={() => setZoomLevel(zoomLevel + 1)}>+</button>
      <button onClick={() => setZoomLevel(zoomLevel - 1)}>-</button>
      <hr />
      <Map zoomLevel={zoomLevel} />
    </>
  );
}
```

```js Map.js active
import { useRef, useEffect } from 'react';
import { MapWidget } from './map-widget.js';

export default function Map({ zoomLevel }) {
  const containerRef = useRef(null);
  const mapRef = useRef(null);

  useEffect(() => {
    if (mapRef.current === null) {
      mapRef.current = new MapWidget(containerRef.current);
    }

    const map = mapRef.current;
    map.setZoom(zoomLevel);
  }, [zoomLevel]);

  return (
    <div
      style={{ width: 200, height: 200 }}
      ref={containerRef}
    />
  );
}
```

```js map-widget.js
import 'leaflet/dist/leaflet.css';
import * as L from 'leaflet';

export class MapWidget {
  constructor(domNode) {
    this.map = L.map(domNode, {
      zoomControl: false,
      doubleClickZoom: false,
      boxZoom: false,
      keyboard: false,
      scrollWheelZoom: false,
      zoomAnimation: false,
      touchZoom: false,
      zoomSnap: 0.1
    });
    L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
      maxZoom: 19,
      attribution: '© OpenStreetMap'
    }).addTo(this.map);
    this.map.setView([0, 0], 0);
  }
  setZoom(level) {
    this.map.setZoom(level);
  }
}
```

```css
button { margin: 5px; }
```

</Sandpack>

In this example, a cleanup function is not needed because the `MapWidget` class manages only the DOM node that was passed to it. After the `Map` React component is removed from the tree, both the DOM node dan the `MapWidget` class instance will be automatically garbage-collected by the browser JavaScript engine.

---

### Fetching data with Effects {/*fetching-data-with-effects*/}

You can use an Effect to fetch data for your component. Note that [if you use a framework,](/learn/start-a-new-react-project#production-grade-react-frameworks) using your framework's data fetching mechanism will be a lot more efficient than writing Effects manually.

If you want to fetch data from an Effect manually, your code might look like this:

```js
import { useState, useEffect } from 'react';
import { fetchBio } from './api.js';

export default function Page() {
  const [person, setPerson] = useState('Alice');
  const [bio, setBio] = useState(null);

  useEffect(() => {
    let ignore = false;
    setBio(null);
    fetchBio(person).then(result => {
      if (!ignore) {
        setBio(result);
      }
    });
    return () => {
      ignore = true;
    };
  }, [person]);

  // ...
```

Note the `ignore` variable which is initialized to `false`, dan is set to `true` during cleanup. This ensures [your code doesn't suffer from "race conditions":](https://maxrozen.com/race-conditions-fetching-data-react-with-useeffect) network responses may arrive in a different order than you sent them.

<Sandpack>

```js App.js
import { useState, useEffect } from 'react';
import { fetchBio } from './api.js';

export default function Page() {
  const [person, setPerson] = useState('Alice');
  const [bio, setBio] = useState(null);
  useEffect(() => {
    let ignore = false;
    setBio(null);
    fetchBio(person).then(result => {
      if (!ignore) {
        setBio(result);
      }
    });
    return () => {
      ignore = true;
    }
  }, [person]);

  return (
    <>
      <select value={person} onChange={e => {
        setPerson(e.target.value);
      }}>
        <option value="Alice">Alice</option>
        <option value="Bob">Bob</option>
        <option value="Taylor">Taylor</option>
      </select>
      <hr />
      <p><i>{bio ?? 'Loading...'}</i></p>
    </>
  );
}
```

```js api.js hidden
export async function fetchBio(person) {
  const delay = person === 'Bob' ? 2000 : 200;
  return new Promise(resolve => {
    setTimeout(() => {
      resolve('This is ' + person + '’s bio.');
    }, delay);
  })
}
```

</Sandpack>

You can also rewrite using the [`async` / `await`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) syntax, but you still need to provide a cleanup function:

<Sandpack>

```js App.js
import { useState, useEffect } from 'react';
import { fetchBio } from './api.js';

export default function Page() {
  const [person, setPerson] = useState('Alice');
  const [bio, setBio] = useState(null);
  useEffect(() => {
    async function startFetching() {
      setBio(null);
      const result = await fetchBio(person);
      if (!ignore) {
        setBio(result);
      }
    }

    let ignore = false;
    startFetching();
    return () => {
      ignore = true;
    }
  }, [person]);

  return (
    <>
      <select value={person} onChange={e => {
        setPerson(e.target.value);
      }}>
        <option value="Alice">Alice</option>
        <option value="Bob">Bob</option>
        <option value="Taylor">Taylor</option>
      </select>
      <hr />
      <p><i>{bio ?? 'Loading...'}</i></p>
    </>
  );
}
```

```js api.js hidden
export async function fetchBio(person) {
  const delay = person === 'Bob' ? 2000 : 200;
  return new Promise(resolve => {
    setTimeout(() => {
      resolve('This is ' + person + '’s bio.');
    }, delay);
  })
}
```

</Sandpack>

Writing data fetching directly in Effects gets repetitive dan makes it difficult to add optimizations like caching dan server rendering later. [It's easier to use a custom Hook--either your own or maintained by the community.](/learn/reusing-logic-with-custom-hooks#when-to-use-custom-hooks)

<DeepDive>

#### What are good alternatives to data fetching in Effects? {/*what-are-good-alternatives-to-data-fetching-in-effects*/}

Writing `fetch` calls inside Effects is a [popular way to fetch data](https://www.robinwieruch.de/react-hooks-fetch-data/), especially in fully client-side apps. This is, however, a very manual approach dan it has significant downsides:

- **Effects don't run on the server.** This means that the initial server-rendered HTML will only include a loading state with no data. The client computer will have to download all JavaScript dan render your app only to discover that now it needs to load the data. This is not very efficient.
- **Fetching directly in Effects makes it easy to create "network waterfalls".** You render the parent component, it fetches some data, renders the child components, dan then they start fetching their data. If the network is not very fast, this is significantly slower than fetching all data in parallel.
- **Fetching directly in Effects usually means you don't preload or cache data.** For example, if the component unmounts dan then mounts again, it would have to fetch the data again.
- **It's not very ergonomic.** There's quite a bit of boilerplate code involved when writing `fetch` calls in a way that doesn't suffer from bugs like [race conditions.](https://maxrozen.com/race-conditions-fetching-data-react-with-useeffect)

This list of downsides is not specific to React. It applies to fetching data on mount with any library. Like with routing, data fetching is not trivial to do well, so we recommend the following approaches:

- **If you use a [framework](/learn/start-a-new-react-project#production-grade-react-frameworks), use its built-in data fetching mechanism.** Modern React frameworks have integrated data fetching mechanisms that are efficient dan don't suffer from the above pitfalls.
- **Otherwise, consider using or building a client-side cache.** Popular open source solutions include [React Query](https://react-query.tanstack.com/), [useSWR](https://swr.vercel.app/), dan [React Router 6.4+.](https://beta.reactrouter.com/en/main/start/overview) You can build your own solution too, in which case you would use Effects under the hood but also add logic for deduplicating requests, caching responses, dan avoiding network waterfalls (by preloading data or hoisting data requirements to routes).

You can continue fetching data directly in Effects if neither of these approaches suit you.

</DeepDive>

---

### Specifying reactive dependencies {/*specifying-reactive-dependencies*/}

**Notice that you can't "choose" the dependencies of your Effect.** Every <CodeStep step={2}>reactive value</CodeStep> used by your Effect's code must be declared as a dependency. Your Effect's dependency list is determined by the surrounding code:

```js [[2, 1, "roomId"], [2, 2, "serverUrl"], [2, 5, "serverUrl"], [2, 5, "roomId"], [2, 8, "serverUrl"], [2, 8, "roomId"]]
function ChatRoom({ roomId }) { // This is a reactive value
  const [serverUrl, setServerUrl] = useState('https://localhost:1234'); // This is a reactive value too

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // This Effect reads these reactive values
    connection.connect();
    return () => connection.disconnect();
  }, [serverUrl, roomId]); // ✅ So you must specify them as dependencies of your Effect
  // ...
}
```

If either `serverUrl` or `roomId` change, your Effect will reconnect to the chat using the new values.

**[Reactive values](/learn/lifecycle-of-reactive-effects#effects-react-to-reactive-values) include props dan all variables dan functions declared directly inside of your component.** Since `roomId` dan `serverUrl` are reactive values, you can't remove them from the dependencies. If you try to omit them dan [your linter is correctly configured for React,](/learn/editor-setup#linting) the linter will flag this as a mistake you need to fix:

```js {8}
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');
  
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // 🔴 React Hook useEffect has missing dependencies: 'roomId' dan 'serverUrl'
  // ...
}
```

**To remove a dependency, you need to ["prove" to the linter *doesn't need* to be a dependency.](/learn/removing-effect-dependencies#removing-unnecessary-dependencies)** For example, you can move `serverUrl` out of your component to prove that it's not reactive dan won't change on re-renders:

```js {1,8}
const serverUrl = 'https://localhost:1234'; // Not a reactive value anymore

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ All dependencies declared
  // ...
}
```

Now that `serverUrl` is not a reactive value (and can't change on a re-render), it doesn't need to be a dependency. **If your Effect's code doesn't use any reactive values, its dependency list should be empty (`[]`):**

```js {1,2,9}
const serverUrl = 'https://localhost:1234'; // Not a reactive value anymore
const roomId = 'music'; // Not a reactive value anymore

function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // ✅ All dependencies declared
  // ...
}
```

[An Effect with empty dependencies](/learn/lifecycle-of-reactive-effects#what-an-effect-with-empty-dependencies-means) doesn't re-run when any of your component's props or state change.

<Pitfall>

If you have an existing codebase, you might have some Effects that suppress the linter like this:

```js {3-4}
useEffect(() => {
  // ...
  // 🔴 Avoid suppressing the linter like this:
  // eslint-ignore-next-line react-hooks/exhaustive-deps
}, []);
```

**When dependencies don't match the code, there is a high risk of introducing bugs.** By suppressing the linter, you "lie" to React about the values your Effect depends on. [Instead, prove they're unnecessary.](/learn/removing-effect-dependencies#removing-unnecessary-dependencies)

</Pitfall>

<Recipes titleText="Examples of passing reactive dependencies" titleId="examples-dependencies">

#### Passing a dependency array {/*passing-a-dependency-array*/}

If you specify the dependencies, your Effect runs **after the initial render _and_ after re-renders with changed dependencies.**

```js {3}
useEffect(() => {
  // ...
}, [a, b]); // Runs again if a or b are different
```

In the below example, `serverUrl` dan `roomId` are [reactive values,](/learn/lifecycle-of-reactive-effects#effects-react-to-reactive-values) so they both must be specified as dependencies. As a result, selecting a different room in the dropdown or editing the server URL input causes the chat to re-connect. However, since `message` isn't used in the Effect (and so it isn't a dependency), editing the message doesn't re-connect to the chat.

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [serverUrl, roomId]);

  return (
    <>
      <label>
        Server URL:{' '}
        <input
          value={serverUrl}
          onChange={e => setServerUrl(e.target.value)}
        />
      </label>
      <h1>Welcome to the {roomId} room!</h1>
      <label>
        Your message:{' '}
        <input value={message} onChange={e => setMessage(e.target.value)} />
      </label>
    </>
  );
}

export default function App() {
  const [show, setShow] = useState(false);
  const [roomId, setRoomId] = useState('general');
  return (
    <>
      <label>
        Choose the chat room:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
        <button onClick={() => setShow(!show)}>
          {show ? 'Close chat' : 'Open chat'}
        </button>
      </label>
      {show && <hr />}
      {show && <ChatRoom roomId={roomId}/>}
    </>
  );
}
```

```js chat.js
export function createConnection(serverUrl, roomId) {
  // A real implementation would actually connect to the server
  return {
    connect() {
      console.log('✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
    }
  };
}
```

```css
input { margin-bottom: 10px; }
button { margin-left: 5px; }
```

</Sandpack>

<Solution />

#### Passing an empty dependency array {/*passing-an-empty-dependency-array*/}

If your Effect truly doesn't use any reactive values, it will only run **after the initial render.**

```js {3}
useEffect(() => {
  // ...
}, []); // Does not run again (except once in development)
```

**Even with empty dependencies, setup dan cleanup will [run one extra time in development](/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development) to help you find bugs.**


In this example, both `serverUrl` dan `roomId` are hardcoded. Since they're declared outside the component, they are not reactive values, dan so they aren't dependencies. The dependency list is empty, so the Effect doesn't re-run on re-renders.

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';
const roomId = 'music';

function ChatRoom() {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []);

  return (
    <>
      <h1>Welcome to the {roomId} room!</h1>
      <label>
        Your message:{' '}
        <input value={message} onChange={e => setMessage(e.target.value)} />
      </label>
    </>
  );
}

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(!show)}>
        {show ? 'Close chat' : 'Open chat'}
      </button>
      {show && <hr />}
      {show && <ChatRoom />}
    </>
  );
}
```

```js chat.js
export function createConnection(serverUrl, roomId) {
  // A real implementation would actually connect to the server
  return {
    connect() {
      console.log('✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
    }
  };
}
```

</Sandpack>

<Solution />


#### Passing no dependency array at all {/*passing-no-dependency-array-at-all*/}

If you pass no dependency array at all, your Effect runs **after every single render (and re-render)** of your component.

```js {3}
useEffect(() => {
  // ...
}); // Always runs again
```

In this example, the Effect re-runs when you change `serverUrl` dan `roomId`, which is sensible. However, it *also* re-runs when you change the `message`, which is probably undesirable. This is why usually you'll specify the dependency array.

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }); // No dependency array at all

  return (
    <>
      <label>
        Server URL:{' '}
        <input
          value={serverUrl}
          onChange={e => setServerUrl(e.target.value)}
        />
      </label>
      <h1>Welcome to the {roomId} room!</h1>
      <label>
        Your message:{' '}
        <input value={message} onChange={e => setMessage(e.target.value)} />
      </label>
    </>
  );
}

export default function App() {
  const [show, setShow] = useState(false);
  const [roomId, setRoomId] = useState('general');
  return (
    <>
      <label>
        Choose the chat room:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
        <button onClick={() => setShow(!show)}>
          {show ? 'Close chat' : 'Open chat'}
        </button>
      </label>
      {show && <hr />}
      {show && <ChatRoom roomId={roomId}/>}
    </>
  );
}
```

```js chat.js
export function createConnection(serverUrl, roomId) {
  // A real implementation would actually connect to the server
  return {
    connect() {
      console.log('✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
    }
  };
}
```

```css
input { margin-bottom: 10px; }
button { margin-left: 5px; }
```

</Sandpack>

<Solution />

</Recipes>

---

### Updating state based on previous state from an Effect {/*updating-state-based-on-previous-state-from-an-effect*/}

When you want to update state based on previous state from an Effect, you might run into a problem:

```js {6,9}
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const intervalId = setInterval(() => {
      setCount(count + 1); // You want to increment the counter every second...
    }, 1000)
    return () => clearInterval(intervalId);
  }, [count]); // 🚩 ... but specifying `count` as a dependency always resets the interval.
  // ...
}
```

Since `count` is a reactive value, it must be specified in the list of dependencies. However, that causes the Effect to cleanup dan setup again every time the `count` changes. This is not ideal. 

To fix this, [pass the `c => c + 1` state updater](/reference/react/useState#updating-state-based-on-the-previous-state) to `setCount`:

<Sandpack>

```js
import { useState, useEffect } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const intervalId = setInterval(() => {
      setCount(c => c + 1); // ✅ Pass a state updater
    }, 1000);
    return () => clearInterval(intervalId);
  }, []); // ✅ Now count is not a dependency

  return <h1>{count}</h1>;
}
```

```css
label {
  display: block;
  margin-top: 20px;
  margin-bottom: 20px;
}

body {
  min-height: 150px;
}
```

</Sandpack>

Now that you're passing `c => c + 1` instead of `count + 1`, [your Effect no longer needs to depend on `count`.](/learn/removing-effect-dependencies#are-you-reading-some-state-to-calculate-the-next-state) As a result of this fix, it won't need to cleanup dan setup the interval again every time the `count` changes.

---


### Removing unnecessary object dependencies {/*removing-unnecessary-object-dependencies*/}

If your Effect depends on an object or a function created during rendering, it might run too often. For example, this Effect re-connects after every render because the `options` object is [different for every render:](/learn/removing-effect-dependencies#does-some-reactive-value-change-unintentionally)

```js {6-9,12,15}
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  const options = { // 🚩 This object is created from scratch on every re-render
    serverUrl: serverUrl,
    roomId: roomId
  };

  useEffect(() => {
    const connection = createConnection(options); // It's used inside the Effect
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // 🚩 As a result, these dependencies are always different on a re-render
  // ...
```

Avoid using an object created during rendering as a dependency. Instead, create the object inside the Effect:

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return (
    <>
      <h1>Welcome to the {roomId} room!</h1>
      <input value={message} onChange={e => setMessage(e.target.value)} />
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  return (
    <>
      <label>
        Choose the chat room:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <hr />
      <ChatRoom roomId={roomId} />
    </>
  );
}
```

```js chat.js
export function createConnection({ serverUrl, roomId }) {
  // A real implementation would actually connect to the server
  return {
    connect() {
      console.log('✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

Now that you create the `options` object inside the Effect, the Effect itself only depends on the `roomId` string.

With this fix, typing into the input doesn't reconnect the chat. Unlike an object which gets re-created, a string like `roomId` doesn't change unless you set it to another value. [Read more about removing dependencies.](/learn/removing-effect-dependencies)

---

### Removing unnecessary function dependencies {/*removing-unnecessary-function-dependencies*/}

If your Effect depends on an object or a function created during rendering, it might run too often. For example, this Effect re-connects after every render because the `createOptions` function is [different for every render:](/learn/removing-effect-dependencies#does-some-reactive-value-change-unintentionally)

```js {4-9,12,16}
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  function createOptions() { // 🚩 This function is created from scratch on every re-render
    return {
      serverUrl: serverUrl,
      roomId: roomId
    };
  }

  useEffect(() => {
    const options = createOptions(); // It's used inside the Effect
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect();
  }, [createOptions]); // 🚩 As a result, these dependencies are always different on a re-render
  // ...
```

By itself, creating a function from scratch on every re-render is not a problem. You don't need to optimize that. However, if you use it as a dependency of your Effect, it will cause your Effect to re-run after every re-render.

Avoid using a function created during rendering as a dependency. Instead, declare it inside the Effect:

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    function createOptions() {
      return {
        serverUrl: serverUrl,
        roomId: roomId
      };
    }

    const options = createOptions();
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return (
    <>
      <h1>Welcome to the {roomId} room!</h1>
      <input value={message} onChange={e => setMessage(e.target.value)} />
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  return (
    <>
      <label>
        Choose the chat room:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <hr />
      <ChatRoom roomId={roomId} />
    </>
  );
}
```

```js chat.js
export function createConnection({ serverUrl, roomId }) {
  // A real implementation would actually connect to the server
  return {
    connect() {
      console.log('✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

Now that you define the `createOptions` function inside the Effect, the Effect itself only depends on the `roomId` string. With this fix, typing into the input doesn't reconnect the chat. Unlike a function which gets re-created, a string like `roomId` doesn't change unless you set it to another value. [Read more about removing dependencies.](/learn/removing-effect-dependencies)

---

### Reading the latest props dan state from an Effect {/*reading-the-latest-props-and-state-from-an-effect*/}

<Wip>

This section describes an **experimental API that has not yet been released** in a stable version of React.

</Wip>

By default, when you read a reactive value from an Effect, you have to add it as a dependency. This ensures that your Effect "reacts" to every change of that value. For most dependencies, that's the behavior you want.

**However, sometimes you'll want to read the *latest* props dan state from an Effect without "reacting" to them.** For example, imagine you want to log the number of the items in the shopping cart for every page visit:

```js {3}
function Page({ url, shoppingCart }) {
  useEffect(() => {
    logVisit(url, shoppingCart.length);
  }, [url, shoppingCart]); // ✅ All dependencies declared
  // ...
}
```

**What if you want to log a new page visit after every `url` change, but *not* if only the `shoppingCart` changes?** You can't exclude `shoppingCart` from dependencies without breaking the [reactivity rules.](#specifying-reactive-dependencies) However, you can express that you *don't want* a piece of code to "react" to changes even though it is called from inside an Effect. [Declare an *Effect Event*](/learn/separating-events-from-effects#declaring-an-effect-event) with the [`useEffectEvent`](/reference/react/experimental_useEffectEvent) Hook, dan move the code reading `shoppingCart` inside of it:

```js {2-4,7,8}
function Page({ url, shoppingCart }) {
  const onVisit = useEffectEvent(visitedUrl => {
    logVisit(visitedUrl, shoppingCart.length)
  });

  useEffect(() => {
    onVisit(url);
  }, [url]); // ✅ All dependencies declared
  // ...
}
```

**Effect Events are not reactive dan must always be omitted from dependencies of your Effect.** This is what lets you put non-reactive code (where you can read the latest value of some props dan state) inside of them. By reading `shoppingCart` inside of `onVisit`, you ensure that `shoppingCart` won't re-run your Effect.

[Read more about how Effect Events let you separate reactive dan non-reactive code.](/learn/separating-events-from-effects#reading-latest-props-and-state-with-effect-events)


---

### Displaying different content on the server dan the client {/*displaying-different-content-on-the-server-and-the-client*/}

If your app uses server rendering (either [directly](/reference/react-dom/server) or via a [framework](/learn/start-a-new-react-project#production-grade-react-frameworks)), your component will render in two different environments. On the server, it will render to produce the initial HTML. On the client, React will run the rendering code again so that it can attach your event handlers to that HTML. This is why, for [hydration](/reference/react-dom/client/hydrateRoot#hydrating-server-rendered-html) to work, your initial render output must be identical on the client dan the server.

In rare cases, you might need to display different content on the client. For example, if your app reads some data from [`localStorage`](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage), it can't possibly do that on the server. Here is how you could implement this:

```js
function MyComponent() {
  const [didMount, setDidMount] = useState(false);

  useEffect(() => {
    setDidMount(true);
  }, []);

  if (didMount) {
    // ... return client-only JSX ...
  }  else {
    // ... return initial JSX ...
  }
}
```

While the app is loading, the user will see the initial render output. Then, when it's loaded dan hydrated, your Effect will run dan set `didMount` to `true`, triggering a re-render. This will switch to the client-only render output. Effects don't run on the server, so this is why `didMount` was `false` during the initial server render.

Use this pattern sparingly. Keep in mind that users with a slow connection will see the initial content for quite a bit of time--potentially, many seconds--so you don't want to make jarring changes to your component's appearance. In many cases, you can avoid the need for this by conditionally showing different things with CSS.

---

## Troubleshooting {/*troubleshooting*/}

### My Effect runs twice when the component mounts {/*my-effect-runs-twice-when-the-component-mounts*/}

When Strict Mode is on, in development, React runs setup dan cleanup one extra time before the actual setup.

This is a stress-test that verifies your Effect’s logic is implemented correctly. If this causes visible issues, your cleanup function is missing some logic. The cleanup function should stop or undo whatever the setup function was doing. The rule of thumb is that the user shouldn’t be able to distinguish between the setup being called once (as in production) dan a setup → cleanup → setup sequence (as in development).

Read more about [how this helps find bugs](/learn/synchronizing-with-effects#step-3-add-cleanup-if-needed) dan [how to fix your logic.](/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development)

---

### My Effect runs after every re-render {/*my-effect-runs-after-every-re-render*/}

First, check that you haven't forgotten to specify the dependency array:

```js {3}
useEffect(() => {
  // ...
}); // 🚩 No dependency array: re-runs after every render!
```

If you've specified the dependency array but your Effect still re-runs in a loop, it's because one of your dependencies is different on every re-render.

You can debug this problem by manually logging your dependencies to the console:

```js {5}
  useEffect(() => {
    // ..
  }, [serverUrl, roomId]);

  console.log([serverUrl, roomId]);
```

You can then right-click on the arrays from different re-renders in the console dan select "Store as a global variable" for both of them. Assuming the first one got saved as `temp1` dan the second one got saved as `temp2`, you can then use the browser console to check whether each dependency in both arrays is the same:

```js
Object.is(temp1[0], temp2[0]); // Is the first dependency the same between the arrays?
Object.is(temp1[1], temp2[1]); // Is the second dependency the same between the arrays?
Object.is(temp1[2], temp2[2]); // ... dan so on for every dependency ...
```

When you find the dependency that is different on every re-render, you can usually fix it in one of these ways:

- [Updating state based on previous state from an Effect](#updating-state-based-on-previous-state-from-an-effect)
- [Removing unnecessary object dependencies](#removing-unnecessary-object-dependencies)
- [Removing unnecessary function dependencies](#removing-unnecessary-function-dependencies)
- [Reading the latest props dan state from an Effect](#reading-the-latest-props-and-state-from-an-effect)

As a last resort (if these methods didn't help), wrap its creation with [`useMemo`](/reference/react/useMemo#memoizing-a-dependency-of-another-hook) or [`useCallback`](/reference/react/useCallback#preventing-an-effect-from-firing-too-often) (for functions).

---

### My Effect keeps re-running in an infinite cycle {/*my-effect-keeps-re-running-in-an-infinite-cycle*/}

If your Effect runs in an infinite cycle, these two things must be true:

- Your Effect is updating some state.
- That state leads to a re-render, which causes the Effect's dependencies to change.

Before you start fixing the problem, ask yourself whether your Effect is connecting to some external system (like DOM, network, a third-party widget, dan so on). Why does your Effect need to set state? Does it synchronize with that external system? Or are you trying to manage your application's data flow with it?

If there is no external system, consider whether [removing the Effect altogether](/learn/you-might-not-need-an-effect) would simplify your logic.

If you're genuinely synchronizing with some external system, think about why dan under what conditions your Effect should update the state. Has something changed that affects your component's visual output? If you need to keep track of some data that isn't used by rendering, a [ref](/reference/react/useRef#referencing-a-value-with-a-ref) (which doesn't trigger re-renders) might be more appropriate. Verify your Effect doesn't update the state (and trigger re-renders) more than needed.

Finally, if your Effect is updating the state at the right time, but there is still a loop, it's because that state update leads to one of the Effect's dependencies changing. [Read how to debug dependency changes.](/reference/react/useEffect#my-effect-runs-after-every-re-render)

---

### My cleanup logic runs even though my component didn't unmount {/*my-cleanup-logic-runs-even-though-my-component-didnt-unmount*/}

The cleanup function runs not only during unmount, but before every re-render with changed dependencies. Additionally, in development, React [runs setup+cleanup one extra time immediately after component mounts.](#my-effect-runs-twice-when-the-component-mounts)

If you have kode cleanup without corresponding kode setup, it's usually a code smell:

```js {2-5}
useEffect(() => {
  // 🔴 Avoid: Cleanup logic without corresponding setup logic
  return () => {
    doSomething();
  };
}, []);
```

Your cleanup logic should be "symmetrical" to the setup logic, dan should stop or undo whatever setup did:

```js {2-3,5}
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [serverUrl, roomId]);
```

[Learn how the Effect lifecycle is different from the component's lifecycle.](/learn/lifecycle-of-reactive-effects#the-lifecycle-of-an-effect)

---

### My Effect does something visual, dan I see a flicker before it runs {/*my-effect-does-something-visual-and-i-see-a-flicker-before-it-runs*/}

If your Effect must block the browser from [painting the screen,](/learn/render-and-commit#epilogue-browser-paint) replace `useEffect` with [`useLayoutEffect`](/reference/react/useLayoutEffect). Note that **this shouldn't be needed for the vast majority of Effects.** You'll only need this if it's crucial to run your Effect before the browser paint: for example, to measure dan position a tooltip before the user sees it.
