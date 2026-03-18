
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>La Verduriza</title>

<style>
body { font-family: Arial; margin: 0; background: #f4f4f4; }
header { background: #2e7d32; color: white; padding: 20px; text-align: center; }
.container { padding: 40px; }

.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 15px;
}

.card {
  background: white;
  padding: 15px;
  border-radius: 10px;
  box-shadow: 0 2px 5px rgba(0,0,0,0.2);
  text-align: center;
}

/* NUEVO */
.product-img {
  width: 100%;
  height: 120px;
  object-fit: cover;
  border-radius: 10px;
  margin-bottom: 10px;
}

input, textarea, select {
  width: 100%;
  padding: 8px;
  margin-top: 5px;
  border-radius: 5px;
  border: 1px solid #ccc;
}

button {
  background: #43a047;
  color: white;
  border: none;
  padding: 8px;
  border-radius: 5px;
  cursor: pointer;
  margin-top: 5px;
}

button:hover { background: #2e7d32; }

.remove-btn {
  background: red;
}

#cartBox {
  position: fixed;
  bottom: 20px;
  right: 20px;
  background: #ff9800;
  padding: 15px;
  border-radius: 15px;
  cursor: pointer;
  color: white;
  font-weight: bold;
}

/* MODAL */
#modal {
  display: none;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0,0,0,0.6);
  justify-content: center;
  align-items: center;
}

.modal-content {
  background: white;
  padding: 20px;
  border-radius: 10px;
  width: 95%;
  max-width: 420px;
  max-height: 90vh;
  overflow-y: auto;
}

.close {
  float: right;
  cursor: pointer;
  color: red;
  font-weight: bold;
}
</style>
</head>

<body>

<header>
<h1>🥬 La Verduriza</h1>
<p>Frutas y verduras frescas hasta tu puerta</p>
</header>

<div class="container">
<h2>Productos</h2>
<div class="grid" id="products"></div>
</div>

<div id="cartBox" onclick="openModal()">
🛒 Ver carrito ($<span id="total">0</span>)
</div>

<!-- MODAL -->
<div id="modal">
<div class="modal-content">

<span class="close" onclick="closeModal()">X</span>

<h2>🧾 Tu pedido</h2>
<div id="cartList"></div>

<hr>

<h3>Datos de entrega</h3>

<label>Nombre:</label>
<input type="text" id="name">

<label>Dirección:</label>
<input type="text" id="address">

<label>Referencias:</label>
<textarea id="references"></textarea>

<label>Método de pago:</label>
<select id="payment">
  <option value="Efectivo">Efectivo</option>
  <option value="Transferencia">Transferencia</option>
</select>

<label>¿Con cuánto paga?</label>
<input type="number" id="cash">

<label>¿No encontraste lo que buscas?</label>
<textarea id="extra" placeholder="Escribe aquí lo que necesitas..."></textarea>

<button onclick="sendWhatsApp()">Confirmar pedido</button>

</div>
</div>

<script>

const products = [
  { name: "Guineo", price: 20, img: "https://uvn-brightspot.s3.amazonaws.com/assets/vixes/imj/1/106401731.jpg" },
  { name: "Plátano", price: 22, img: "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTndFDmfikZJMF6qeCEWuGVPmkTY0z76rLtgg&s" },
  { name: "Naranja", price: 27, img: "https://frutas.consumer.es/sites/frutas/files/styles/pagina_cabecera_desktop/public/2025-04/naranja.webp?h=e5bbb8f9&itok=0rigDt3S" },
  { name: "Tomate", price: 42, img: "https://agrichem.mx/wp-content/uploads/2017/02/tomate2.jpg" },
  { name: "Cebolla", price: 20, img: "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQFoN9386WFngYKhLQG9mSOTqumjeVJs6Ckkw&s" },
  { name: "Zanahoria", price: 22, img: "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQsQ4D1t2mJA1i6sQC2hghLKu-Bzn_FXZ9v4Q&s" }
];

let cart = [];
let total = 0;

const container = document.getElementById("products");

products.forEach((p, i) => {
  const div = document.createElement("div");
  div.className = "card";

  div.innerHTML = `
    <img src="${p.img}" class="product-img">
    <h3>${p.name}</h3>
    <p>$${p.price} / kg</p>
    <input type="number" id="qty${i}" min="1" value="1"> kg
    <button onclick="addToCart(${i})">Agregar</button>
  `;

  container.appendChild(div);
});

function addToCart(i) {
  const qty = document.getElementById(`qty${i}`).value;
  if (qty <= 0) return;

  const product = products[i];
  const subtotal = product.price * qty;

  cart.push({
    name: product.name,
    qty: qty,
    subtotal: subtotal
  });

  total += subtotal;
  updateTotal();

  alert(`${product.name} agregado`);
}

function updateTotal() {
  document.getElementById("total").innerText = total;
}

function removeFromCart(index) {
  total -= cart[index].subtotal;
  cart.splice(index, 1);
  renderCart();
  updateTotal();
}

function renderCart() {
  const list = document.getElementById("cartList");
  list.innerHTML = "";

  if (cart.length === 0) {
    list.innerHTML = "<p>Carrito vacío</p>";
    return;
  }

  cart.forEach((item, i) => {
    list.innerHTML += `
      <div>
        ${item.name} - ${item.qty} kg = $${item.subtotal}
        <button class="remove-btn" onclick="removeFromCart(${i})">X</button>
      </div>
    `;
  });
}

function openModal() {
  renderCart();

  if (cart.length === 0) {
    alert("El carrito está vacío");
    return;
  }

  document.getElementById("modal").style.display = "flex";
}

function closeModal() {
  document.getElementById("modal").style.display = "none";
}

function sendWhatsApp() {

  const name = document.getElementById("name").value.trim();
  const address = document.getElementById("address").value.trim();
  const references = document.getElementById("references").value.trim();
  const payment = document.getElementById("payment").value;
  const cash = document.getElementById("cash").value;
  const extra = document.getElementById("extra").value.trim();

  if (!name || !address) {
    alert("Completa nombre y dirección");
    return;
  }

  let message = "🛒 *Pedido - La Verduriza* %0A%0A";

  message += `👤 ${name}%0A📍 ${address}%0A📝 ${references}%0A`;
  message += `💳 Pago: ${payment}%0A`;

  if (payment === "Efectivo" && cash) {
    message += `💵 Paga con: $${cash}%0A`;
  }

  message += "%0A🧺 *Productos:* %0A";

  cart.forEach(item => {
    message += `• ${item.name} - ${item.qty} kg = $${item.subtotal}%0A`;
  });

  if (extra) {
    message += `%0A❓ También busco:%0A${extra}%0A`;
  }

  message += `%0ATotal: $${total}`;

  const phone = "5219613267670";
  const url = `https://wa.me/${phone}?text=${message}`;

  window.open(url, "_blank");
}

</script>

</body>
</html>
