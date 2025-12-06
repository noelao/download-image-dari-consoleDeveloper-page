# #readme <tutorial>

## Langkah 1: 
- Pilih Elemen TargetBuka website target.
- Klik Kanan pada area/div/container yang berisi gambar-gambar tersebut $\rightarrow$ Inspect (Inspeksi).
- Di panel Elements, pastikan elemen pembungkus (div/section) tersebut tersorot biru.
- Perhatikan di sebelah elemen tersebut biasanya ada tulisan == $0.Ini artinya: Chrome/Edge menyimpan elemen yang sedang kamu sorot ke dalam variabel bernama $0.
  
## Langkah 2: 
- Paste Script "Penyedot" Gambar HDBuka tab Console, lalu copy-paste kode sakti di bawah ini.Kode ini sudah saya lengkapi dengan logika untuk mencari resolusi tertinggi dari srcset atau link induknya.

  ``` javascript
  (async function downloadAllImages() {
    // 1. Ambil elemen yang sedang kamu inspect ($0)
    const container = $0; 
    
    if (!container) {
        console.error("❌ Eits! Kamu belum memilih elemen. Klik kanan elemen di tab 'Elements' sampai muncul '$0'.");
        return;
    }

    // 2. Cari semua tag <img> di dalam elemen tersebut
    const images = container.querySelectorAll('img');
    console.log(`🔍 Ditemukan ${images.length} gambar. Sedang memproses resolusi tertinggi...`);

    // Fungsi Helper: Mencari URL terbesar dari srcset
    function getHighResUrl(img) {
        // Cek 1: Apakah punya srcset? (Biasanya format: "url1 500w, url2 1000w")
        if (img.srcset) {
            const sources = img.srcset.split(',').map(src => {
                const parts = src.trim().split(' ');
                return {
                    url: parts[0],
                    width: parts[1] ? parseInt(parts[1].replace('w', '')) : 0
                };
            });
            // Urutkan dari width terbesar ke terkecil, ambil yang pertama
            sources.sort((a, b) => b.width - a.width);
            if (sources.length > 0) return sources[0].url;
        }

        // Cek 2: Apakah dibungkus <a> yang mengarah ke file gambar? (Biasanya lightbox)
        const parent = img.closest('a');
        if (parent && parent.href && parent.href.match(/\.(jpg|jpeg|png|webp|gif)/i)) {
            return parent.href;
        }

        // Cek 3: Fallback ke src biasa (kadang src adalah thumbnail)
        // Cek apakah ada atribut data-src atau data-full (biasa dipakai lazyload)
        return img.dataset.src || img.dataset.full || img.dataset.highres || img.src;
    }

    // Fungsi Helper: Download File
    async function downloadImage(url, index) {
        try {
            const response = await fetch(url);
            const blob = await response.blob();
            const blobUrl = URL.createObjectURL(blob);
            
            const link = document.createElement('a');
            link.href = blobUrl;
            
            // Tebak ekstensi file
            let ext = url.split('.').pop().split('?')[0];
            if (ext.length > 4 || ext.length < 3) ext = 'jpg'; // Default jika ekstensi aneh
            
            link.download = `gambar_hd_${index + 1}.${ext}`;
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);
            URL.revokeObjectURL(blobUrl);
            console.log(`✅ Terdownload: ${url}`);
        } catch (err) {
            console.warn(`⚠️ Gagal download langsung (CORS Protected), mencoba buka tab baru: ${url}`);
            window.open(url, '_blank');
        }
    }

    // 3. Loop dan Eksekusi dengan jeda (agar browser tidak crash/diblokir)
    let count = 0;
    for (const img of images) {
        const hdUrl = getHighResUrl(img);
        if (hdUrl) {
            await downloadImage(hdUrl, count);
            count++;
            // Beri jeda 500ms (setengah detik) per gambar agar sopan ke server
            await new Promise(r => setTimeout(r, 500)); 
        }
    }

    console.log("🎉 Selesai! Semua gambar berhasil diunduh.");
})();
  ```
