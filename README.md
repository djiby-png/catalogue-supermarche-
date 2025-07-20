# catalogue-supermarche-
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Catalogue Supermarché</title>
  <link rel="manifest" href="manifest.json">
  <style>
    body { font-family: Arial; padding: 20px; background-color: #f0f0f0; }
    input, button { padding: 10px; margin: 5px 0; width: 100%; box-sizing: border-box; }
    .product { padding: 10px; background: white; border: 1px solid #ccc; margin-bottom: 5px; }
    #scanner { width: 100%; max-height: 300px; }
  </style>
</head>
<body>

<h2>🛒 Catalogue Supermarché</h2>

<input type="text" id="search" placeholder="🔍 Rechercher un produit..." oninput="searchProduct()" />
<button onclick="startScanner()">📷 Scanner un code-barres</button>
<video id="scanner" style="display:none;"></video>

<div id="product-list"></div>

<hr />
<h3>➕ Ajouter un produit</h3>
<input type="text" id="newName" placeholder="Nom du produit" />
<input type="number" id="newPrice" placeholder="Prix (FCFA)" />
<input type="text" id="newCode" placeholder="Code-barres (EAN)" />
<button onclick="addProduct()">Ajouter</button>

<script src="https://unpkg.com/@ericblade/quagga2/dist/quagga.min.js"></script>
<script>
  let products = JSON.parse(localStorage.getItem("products")) || [
    { code: "123456789012", name: "Bouteille d'eau", price: 300 },
    { code: "987654321098", name: "Riz 5kg", price: 4000 },
    { code: "456789123456", name: "Savon", price: 200 }
  ];

  const listDiv = document.getElementById("product-list");

  function displayProducts(filtered = products) {
    listDiv.innerHTML = '';
    filtered.forEach(p => {
      listDiv.innerHTML += `<div class="product"><strong>${p.name}</strong> – ${p.price} FCFA</div>`;
    });
  }

  function searchProduct() {
    const q = document.getElementById("search").value.toLowerCase();
    const filtered = products.filter(p => p.name.toLowerCase().includes(q));
    displayProducts(filtered);
  }

  function addProduct() {
    const name = document.getElementById("newName").value.trim();
    const price = parseInt(document.getElementById("newPrice").value);
    const code = document.getElementById("newCode").value.trim();

    if (!name || !price || !code) {
      alert("Remplis tous les champs !");
      return;
    }

    products.push({ name, price, code });
    localStorage.setItem("products", JSON.stringify(products));
    displayProducts();
    document.getElementById("newName").value = "";
    document.getElementById("newPrice").value = "";
    document.getElementById("newCode").value = "";
    alert("Produit ajouté ✅");
  }

  function startScanner() {
    const scanner = document.getElementById("scanner");
    scanner.style.display = "block";
    Quagga.init({
      inputStream: {
        type: "LiveStream",
        target: scanner,
        constraints: { facingMode: "environment" }
      },
      decoder: { readers: ["ean_reader"] }
    }, function(err) {
      if (err) {
        alert("Erreur : " + err);
        return;
      }
      Quagga.start();
    });

    Quagga.onDetected(data => {
      const code = data.codeResult.code;
      const product = products.find(p => p.code === code);
      if (product) {
        alert(`✅ ${product.name} – ${product.price} FCFA`);
      } else {
        alert(`❌ Produit non trouvé pour le code : ${code}`);
      }
      Quagga.stop();
      scanner.style.display = "none";
    });
  }

  displayProducts();
</script>

<script>
  if ('serviceWorker' in navigator) {
    navigator.serviceWorker.register('sw.js')
      .then(() => console.log('Service Worker enregistré ✅'));
  }
</script>

</body>
</html>
{
  "name": "Catalogue Supermarché",
  "short_name": "Supermarché",
  "start_url": "index.html",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#007bff",
  "icons": [
    {
      "src": "icon.png",
      "sizes": "192x192",
      "type": "image/png"
    }
  ]
}
self.addEventListener('install', e => {
  e.waitUntil(
    caches.open('app-cache').then(cache => {
      return cache.addAll([
        'index.html',
        'manifest.json',
        'sw.js',
        'icon.png'
      ]);
    })
  );
});

self.addEventListener('fetch', e => {
  e.respondWith(
    caches.match(e.request).then(response => response || fetch(e.request))
  );
});
