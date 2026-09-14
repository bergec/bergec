# Bergeç — içerik düzenleme rehberi

Site parola korumalı olarak yayınlanıyor: her sayfa istemci tarafında AES-256-GCM ile şifrelenmiş durumda, tarayıcı doğru parolayı girene kadar hiçbir metin okunamaz. Bu yüzden dosyaları **iki kopya** olarak tutuyoruz:

- `_source_plaintext_DO_NOT_PUBLISH/` — düzenlenebilir, şifresiz asıl kaynaklar. **Bu klasör `.gitignore`'da, asla GitHub'a gönderilmez.**
- Repo kökündeki `*.html` dosyaları (`index.html`, `godel-mektubu.html`, `enigma-1.html` vb.) — yukarıdakilerin şifrelenmiş, yayınlanan hâli. **Bunları elle düzenlemeyin**, script her seferinde yeniden üretir.

## Bir yazıyı düzenlemek

1. `_source_plaintext_DO_NOT_PUBLISH/<dosya>.html` içindeki metni normal bir metin editörüyle düzenleyin (düz HTML — başlıklar `<h2>`, paragraflar `<p>`, matematik `<span class="im">...</span>` veya `<div class="eq eq-block">$$...$$</div>` içinde LaTeX).

   **Dikkat:** Metin veya formül içinde literal `<` ya da `>` kullanmayın (örn. "i<j", "n>0") — tarayıcı bunu HTML etiketi sanıp sayfayı bozar. Bunun yerine `&lt;` ve `&gt;` yazın.

2. Değişikliği şifreleyip yayınlanacak dosyayı güncelleyin:

   ```bash
   cd "Bergeç nostalgic website redesign"
   node tools/encrypt.js "PAROLA" "_source_plaintext_DO_NOT_PUBLISH/<dosya>.html" "<dosya>.html"
   ```

   Parola şu an: `QH3S9ZtPGWGuh9z7`

3. Yerelde kontrol edin (isteğe bağlı ama önerilir):

   ```bash
   python3 -m http.server 8000
   ```
   sonra tarayıcıda `http://127.0.0.1:8000/<dosya>.html` açıp parolayı girin.

4. Gönderin:

   ```bash
   git add <dosya>.html
   git commit -m "..."
   git push
   ```

   Push'tan ~30 saniye sonra `https://bergec.github.io/` üzerinde güncellenir.

## Yeni bir yazı eklemek

1. `_source_plaintext_DO_NOT_PUBLISH/` içine, mevcut bir makaleyi (örn. `godel-mektubu.html`) kopyalayıp şablon olarak kullanın — aynı CSS sınıfları ve KaTeX script'i zaten içinde.
2. İçeriği yazın, yukarıdaki `<`/`>` kuralına dikkat edin.
3. `node tools/encrypt.js "PAROLA" "_source_plaintext_DO_NOT_PUBLISH/yeni-yazi.html" "yeni-yazi.html"` ile şifreleyin.
4. Anasayfaya link eklemek için `_source_plaintext_DO_NOT_PUBLISH/index.html` içinde `posts` dizisine (JS array, dosyanın sonlarında `<script data-dc-script>` içinde) yeni bir satır ekleyip index.html'i de aynı şekilde yeniden şifreleyin.
5. Hepsini `git add`, `commit`, `push`.

## Fotoğraf/görsel eklemek

Görseller ayrı dosya olarak DEĞİL, `<img src="data:image/jpeg;base64,...">` şeklinde HTML'in içine gömülüyor — böylece şifrelenince onlar da parola arkasında kalıyor. Bir görseli base64'e çevirip HTML'e gömmek için:

```bash
python3 -c "
from PIL import Image, io, base64
im = Image.open('foto.png').convert('RGB')
if im.width > 820:
    im = im.resize((820, int(im.height*820/im.width)))
im.save('/tmp/foto.jpg', format='JPEG', quality=82)
print(base64.b64encode(open('/tmp/foto.jpg','rb').read()).decode())
" > foto.b64.txt
```

Sonra çıkan metni `<img src="data:image/jpeg;base64,BURAYA_YAPIŞTIR">` şeklinde ilgili `_source_plaintext_DO_NOT_PUBLISH/*.html` dosyasına ekleyip yeniden şifreleyin.

## Parolayı değiştirmek

Tüm sayfaları yeni parolayla yeniden şifrelemeniz gerekir (hepsi aynı parolayı paylaşıyor):

```bash
for f in index godel-mektubu enigma-1 enigma-2 enigma-3 knuth-1 knuth-2; do
  node tools/encrypt.js "YENİ_PAROLA" "_source_plaintext_DO_NOT_PUBLISH/$f.html" "$f.html"
done
git add *.html && git commit -m "Parolayı değiştir" && git push
```

## Dizin yapısı özeti

- `_ds/` — tasarım sistemi (renkler, fontlar, `styles.css`) — bunu değiştirmek tüm sitenin görünümünü etkiler.
- `_source_plaintext_DO_NOT_PUBLISH/` — düzenlenebilir kaynaklar (git'e gönderilmez).
- `tools/encrypt.js` — şifreleme scripti (Node.js, ekstra kurulum gerektirmez).
- `yazılar/` — orijinal PDF makaleler (git'e gönderilmez, yalnızca referans).
- Repo kökündeki `*.html` — yayınlanan, şifreli sayfalar.
