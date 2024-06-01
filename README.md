# Caraku mengoptimalkan performa next.js app.

Berikut adalah penjelasan lebih detail tentang kapan sebaiknya menggunakan setiap teknik optimasi dalam aplikasi Next.js.

### 1. **Penerapan SSR dan SSG dengan Bijak**

- **Server-Side Rendering (SSR)**: Gunakan `getServerSideProps` jika halaman Anda memerlukan data yang selalu terbaru pada setiap permintaan, seperti dashboard pengguna atau halaman detail produk yang sering berubah.
  
  ```jsx
  export async function getServerSideProps(context) {
    const res = await fetch('https://api.example.com/data')
    const data = await res.json()

    return {
      props: { data },
    }
  }
  ```

- **Static Site Generation (SSG)**: Gunakan `getStaticProps` untuk halaman yang datanya tidak berubah sering, seperti halaman blog atau landing page. `getStaticPaths` digunakan untuk mendukung rute dinamis pada halaman statis.
  
  ```jsx
  export async function getStaticProps() {
    const res = await fetch('https://api.example.com/data')
    const data = await res.json()

    return {
      props: { data },
      revalidate: 10, // ISR
    }
  }
  ```

- **Incremental Static Regeneration (ISR)**: Kombinasi dari SSG dan kemampuan untuk memperbarui halaman setelah build tanpa perlu deploy ulang seluruh situs. Gunakan ini jika Anda perlu konten statis yang diperbarui secara berkala.
  
  ```jsx
  export async function getStaticProps() {
    const res = await fetch('https://api.example.com/data')
    const data = await res.json()

    return {
      props: { data },
      revalidate: 60, // revalidate setiap 60 detik
    }
  }
  ```

### 2. **Optimasi Gambar**

- Gunakan komponen `next/image` jika Anda perlu mengoptimalkan gambar secara otomatis, termasuk pengubahan ukuran dan format modern seperti WebP. Ini berguna untuk halaman dengan banyak gambar atau halaman yang perlu performa tinggi.

  ```jsx
  import Image from 'next/image'

  const MyImage = () => (
    <Image
      src="/me.png"
      alt="Picture of the author"
      width={500}
      height={500}
    />
  )
  ```

### 3. **Pemecahan Kode (Code Splitting) dan Lazy Loading**

- Gunakan `dynamic` untuk memuat komponen besar secara lazy jika komponen tersebut tidak perlu dimuat segera pada halaman. Misalnya, komponen peta atau grafik yang tidak langsung terlihat oleh pengguna.

  ```jsx
  import dynamic from 'next/dynamic'

  const DynamicComponent = dynamic(() => import('../components/HeavyComponent'), {
    ssr: false
  })

  const Page = () => (
    <div>
      <DynamicComponent />
    </div>
  )

  export default Page
  ```

### 4. **Optimasi JavaScript dan CSS**

- Minifikasi kode JavaScript dan CSS untuk mengurangi ukuran bundle. Ini berguna untuk semua aplikasi produksi. Secara default, Next.js sudah melakukan ini, tetapi Anda bisa mengoptimalkan lebih lanjut dengan plugin tambahan.

### 5. **Penggunaan CDN**

- Gunakan CDN untuk menyajikan aset statis jika Anda ingin mengurangi latensi dan meningkatkan waktu muat. Ini sangat penting untuk aplikasi global atau dengan banyak aset statis seperti gambar, video, atau file besar.

### 6. **Pengaturan Webpack dan Babel**

- Sesuaikan konfigurasi Webpack untuk menambahkan plugin optimasi tambahan jika Anda perlu performa build yang lebih baik atau menggunakan fitur lanjutan Webpack. Misalnya, untuk caching build atau menggunakan loader khusus.

  ```javascript
  // next.config.js
  module.exports = {
    webpack: (config, { isServer }) => {
      if (!isServer) {
        config.optimization.splitChunks = {
          chunks: 'all',
        };
      }
      return config;
    },
  };
  ```

### 7. **Komponen Reusable dan Memoization**

- Gunakan `React.memo` jika komponen Anda tidak berubah sering dan Anda ingin menghindari rendering ulang yang tidak perlu. Misalnya, komponen tombol atau label yang statis.

  ```jsx
  const MyComponent = React.memo(({ data }) => {
    return <div>{data}</div>
  })
  ```

- Gunakan `useMemo` dan `useCallback` untuk meminimalkan pembuatan ulang fungsi atau nilai yang mahal secara komputasi. Ini berguna untuk komponen dengan logika kompleks atau rendering mahal.

  ```jsx
  const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
  const memoizedCallback = useCallback(() => {
    doSomething(a, b);
  }, [a, b]);
  ```

### 8. **Penggunaan Preact di Produksi**

- Gantikan React dengan Preact jika ukuran bundle menjadi masalah besar. Ini cocok untuk aplikasi yang perlu optimasi ukuran bundle secara ekstrem tanpa banyak fitur lanjutan React.

  ```javascript
  // next.config.js
  module.exports = {
    webpack: (config, { dev, isServer }) => {
      if (!dev && !isServer) {
        Object.assign(config.resolve.alias, {
          'react/jsx-runtime.js': 'preact/compat/jsx-runtime',
          'react/jsx-dev-runtime.js': 'preact/compat/jsx-dev-runtime',
          react: 'preact/compat',
          'react-dom/test-utils': 'preact/test-utils',
          'react-dom': 'preact/compat',
        });
      }
      return config;
    },
  };
  ```

### 9. **Optimasi Font**

- Gunakan `next/font` untuk memuat font secara efisien dan optimalkan preloading jika Anda menggunakan font custom. Ini membantu mengurangi waktu render pertama (FCP) dan memberikan pengalaman pengguna yang lebih baik.

### 10. **Server Configuration**

- Sesuaikan pengaturan server untuk cache response yang tepat jika aplikasi Anda membutuhkan performa tinggi dan data tidak selalu berubah. Misalnya, menggunakan Redis untuk caching data API yang sering diakses.
- Gunakan HTTP/2 dan kompresi gzip atau Brotli untuk mempercepat pengiriman data.

### 11. **Analisis dan Pemantauan**

- Gunakan alat seperti Lighthouse atau PageSpeed Insights untuk menganalisis performa aplikasi Anda. Ini berguna untuk mengidentifikasi bottleneck dan area yang perlu perbaikan.
- Implementasikan monitoring dengan Sentry atau LogRocket untuk mendeteksi dan memperbaiki masalah performa secara real-time.

### Contoh `next.config.js` untuk Optimasi

```javascript
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
});

module.exports = withBundleAnalyzer({
  images: {
    domains: ['example.com'],
  },
  compress: true,
  webpack(config, { dev, isServer }) {
    if (!dev && !isServer) {
      // Enable cache
      config.cache = {
        type: 'filesystem',
        buildDependencies: {
          config: [__filename],
        },
      };
    }
    return config;
  },
});
```

Dengan mengikuti panduan ini, Anda bisa menerapkan optimasi yang tepat sesuai dengan kebutuhan spesifik aplikasi Next.js Anda, memberikan pengalaman pengguna yang cepat dan responsif.
