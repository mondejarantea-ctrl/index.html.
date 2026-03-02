<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>PriceGuard SaaS</title>

<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600&display=swap" rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<style>
body{
  font-family:Inter;
  background:#f4f7f9;
  margin:0;
}
header{
  background:#2563eb;
  color:white;
  padding:20px;
  text-align:center;
}
.container{
  max-width:1000px;
  margin:30px auto;
  background:white;
  padding:20px;
  border-radius:10px;
  box-shadow:0 5px 15px rgba(0,0,0,0.1);
}
input,button{
  padding:8px;
  margin:5px;
  border-radius:5px;
  border:1px solid #ccc;
}
button{
  background:#2563eb;
  color:white;
  border:none;
  cursor:pointer;
}
table{
  width:100%;
  border-collapse:collapse;
}
th,td{
  padding:8px;
  border-bottom:1px solid #ddd;
}
.alert{
  color:red;
  font-weight:bold;
}
.normal{
  color:green;
  font-weight:bold;
}
</style>
</head>

<body>

<header>
<h1>PriceGuard - Monitor Profesional</h1>
</header>

<div class="container">

<h2>Registro / Login</h2>
<input type="email" id="email" placeholder="Email">
<input type="password" id="password" placeholder="Contraseña">
<button onclick="register()">Registrarse</button>
<button onclick="login()">Login</button>
<button onclick="logout()">Logout</button>

<h2>Agregar Producto</h2>
<input type="text" id="productName" placeholder="Producto">
<input type="number" id="myPrice" placeholder="Tu Precio">
<input type="number" id="competitorPrice" placeholder="Precio Competidor">
<button onclick="addProduct()">Guardar</button>

<h2>Productos</h2>
<table id="productsTable"></table>

<h2>Historial</h2>
<canvas id="chart"></canvas>

</div>

<script type="module">

import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
import { getAuth, createUserWithEmailAndPassword, signInWithEmailAndPassword, signOut, onAuthStateChanged } 
from "https://www.gstatic.com/firebasejs/10.7.1/firebase-auth.js";
import { getFirestore, collection, addDoc, query, where, onSnapshot } 
from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

const firebaseConfig = {
  apiKey: "TU_API_KEY",
  authDomain: "priceguard-6da48.firebaseapp.com",
  projectId: "priceguard-6da48",
  storageBucket: "priceguard-6da48.appspot.com",
  messagingSenderId: "916598874837",
  appId: "TU_APP_ID"
};

const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);

let currentUser = null;

// REGISTRO
window.register = function(){
  createUserWithEmailAndPassword(
    auth,
    document.getElementById("email").value,
    document.getElementById("password").value
  );
}

// LOGIN
window.login = function(){
  signInWithEmailAndPassword(
    auth,
    document.getElementById("email").value,
    document.getElementById("password").value
  );
}

// LOGOUT
window.logout = function(){
  signOut(auth);
}

// Detectar usuario activo
onAuthStateChanged(auth, user=>{
  if(user){
    currentUser = user;
    loadProducts();
  }
});

// AGREGAR PRODUCTO
window.addProduct = async function(){
  if(!currentUser){
    alert("Debes iniciar sesión");
    return;
  }

  await addDoc(collection(db,"products"),{
    user: currentUser.uid,
    name: document.getElementById("productName").value,
    myPrice: Number(document.getElementById("myPrice").value),
    competitorPrice: Number(document.getElementById("competitorPrice").value),
    timestamp: new Date()
  });
}

// CARGAR PRODUCTOS
function loadProducts(){

  const q = query(
    collection(db,"products"),
    where("user","==",currentUser.uid)
  );

  onSnapshot(q, snapshot=>{

    let table="<tr><th>Producto</th><th>Mi Precio</th><th>Competidor</th><th>Estado</th></tr>";

    snapshot.forEach(doc=>{
      let data=doc.data();

      let estado = data.competitorPrice < data.myPrice 
      ? "<span class='alert'>Más barato</span>" 
      : "<span class='normal'>OK</span>";

      table += `
      <tr>
        <td>${data.name}</td>
        <td>${data.myPrice}</td>
        <td>${data.competitorPrice}</td>
        <td>${estado}</td>
      </tr>
      `;
    });

    document.getElementById("productsTable").innerHTML=table;
  });
}

// GRÁFICA
const ctx = document.getElementById('chart');

const chart = new Chart(ctx,{
  type:'line',
  data:{
    labels:[],
    datasets:[
      {
        label:'Mi Precio',
        data:[],
        borderColor:'#2563eb'
      },
      {
        label:'Competidor',
        data:[],
        borderColor:'#ef4444'
      }
    ]
  }
});

// Escuchar historial
onSnapshot(collection(db,"products"), snapshot=>{
  chart.data.labels=[];
  chart.data.datasets[0].data=[];
  chart.data.datasets[1].data=[];

  snapshot.forEach(doc=>{
    let d=doc.data();
    chart.data.labels.push(d.name);
    chart.data.datasets[0].data.push(d.myPrice);
    chart.data.datasets[1].data.push(d.competitorPrice);
  });

  chart.update();
});

</script>

</body>
</html>
