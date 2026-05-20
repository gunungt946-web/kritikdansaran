<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>KELURAHAN GUNUNG TINGGI</title>

<style>
*{
    margin:0;
    padding:0;
    box-sizing:border-box;
    font-family:Arial, sans-serif;
}

body{
    background:#f1f5ff;
}

/* HEADER */
header{
    background:linear-gradient(90deg,#004aad,#0077ff);
    color:white;
    padding:15px 20px;
    display:flex;
    justify-content:space-between;
    align-items:center;
}

.running-text{
    width:100%;
    overflow:hidden;
    white-space:nowrap;
}

.running-text h1{
    display:inline-block;
    padding-left:100%;
    animation:jalan 13s linear infinite;
    font-size:28px;
}

@keyframes jalan{
    0%{transform:translateX(0);}
    100%{transform:translateX(-200%);}
}

.login-btn{
    background:white;
    color:#004aad;
    border:none;
    padding:10px 16px;
    border-radius:10px;
    cursor:pointer;
    font-weight:bold;
}

/* FORM */
.container{
    width:90%;
    max-width:700px;
    margin:40px auto;
    background:white;
    padding:25px;
    border-radius:15px;
    box-shadow:0 5px 15px rgba(0,0,0,0.1);
}

input,textarea,select{
    width:100%;
    padding:12px;
    margin-top:12px;
    border-radius:10px;
    border:1px solid #ccc;
}

textarea{
    height:120px;
    resize:none;
}

.rating{
    text-align:center;
    font-size:40px;
    margin-top:15px;
}

.rating span{
    cursor:pointer;
    color:#ccc;
}

.rating span.active{
    color:gold;
}

.kirim{
    width:100%;
    margin-top:15px;
    padding:12px;
    background:#004aad;
    color:white;
    border:none;
    border-radius:10px;
    cursor:pointer;
}

/* ADMIN */
.admin-panel{
    display:none;
    width:95%;
    margin:30px auto;
    background:white;
    padding:20px;
    border-radius:15px;
}

table{
    width:100%;
    border-collapse:collapse;
}

th,td{
    border:1px solid #ddd;
    padding:10px;
    text-align:center;
}

th{
    background:#004aad;
    color:white;
}

.hapus{
    background:red;
    color:white;
    border:none;
    padding:6px 10px;
    border-radius:5px;
    cursor:pointer;
}

/* MODAL */
.modal{
    display:none;
    position:fixed;
    width:100%;
    height:100%;
    background:rgba(0,0,0,0.5);
    justify-content:center;
    align-items:center;
}

.modal-content{
    background:white;
    padding:20px;
    border-radius:10px;
    width:300px;
}
</style>
</head>

<body>

<header>
<div class="running-text">
<h1>SELAMAT DATANG DI WEBSITE KRITIK DAN SARAN KELURAHAN GUNUNG TINGGI</h1>
</div>

<button class="login-btn" onclick="openLogin()">Login Admin</button>
</header>

<div class="container">
<h2>Form Kritik & Saran</h2>

<input id="nama" placeholder="Nama">

<select id="jenis">
<option value="Kritik">Kritik</option>
<option value="Saran">Saran</option>
</select>

<textarea id="pesan" placeholder="Pesan"></textarea>

<div class="rating">
<span onclick="setRating(1)">★</span>
<span onclick="setRating(2)">★</span>
<span onclick="setRating(3)">★</span>
<span onclick="setRating(4)">★</span>
<span onclick="setRating(5)">★</span>
</div>

<button class="kirim" onclick="kirimData()">Kirim</button>
</div>

<div class="admin-panel" id="adminPanel">
<h2>Data Kritik & Saran (PUBLIK)</h2>
<table>
<thead>
<tr>
<th>No</th>
<th>Nama</th>
<th>Jenis</th>
<th>Pesan</th>
<th>Rating</th>
<th>Aksi</th>
</tr>
</thead>
<tbody id="dataTable"></tbody>
</table>
</div>

<!-- LOGIN -->
<div class="modal" id="loginModal">
<div class="modal-content">
<h3>Login Admin</h3>
<input id="username" placeholder="Username">
<input id="password" type="password" placeholder="Password">
<button class="kirim" onclick="loginAdmin()">Login</button>
</div>
</div>

<!-- FIREBASE -->
<script src="https://www.gstatic.com/firebasejs/10.12.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.0/firebase-firestore-compat.js"></script>

<script>
/* 🔥 ISI FIREBASE KAMU DI SINI */
const firebaseConfig = {
    apiKey: "ISI",
    authDomain: "ISI",
    projectId: "ISI",
    storageBucket: "ISI",
    messagingSenderId: "ISI",
    appId: "ISI"
};

firebase.initializeApp(firebaseConfig);
const db = firebase.firestore();

/* AUTO CHECK */
if (!firebase.apps.length) {
    firebase.initializeApp(firebaseConfig);
}

let rating = 0;

/* RATING */
function setRating(r){
    rating = r;
    document.querySelectorAll(".rating span").forEach((el,i)=>{
        el.classList.toggle("active", i < r);
    });
}

/* KIRIM DATA */
function kirimData(){
    let nama = document.getElementById("nama").value;
    let jenis = document.getElementById("jenis").value;
    let pesan = document.getElementById("pesan").value;

    if(!nama || !pesan || rating === 0){
        alert("Lengkapi data!");
        return;
    }

    db.collection("kritik").add({
        nama,
        jenis,
        pesan,
        rating: "⭐".repeat(rating),
        waktu: Date.now()
    }).then(()=>{
        alert("Terkirim!");
        clearForm();
        tampilkanData();
    });
}

/* CLEAR FORM */
function clearForm(){
    document.getElementById("nama").value = "";
    document.getElementById("pesan").value = "";
    rating = 0;
    document.querySelectorAll(".rating span").forEach(e=>e.classList.remove("active"));
}

/* TAMPIL DATA REALTIME */
function tampilkanData(){
    let table = document.getElementById("dataTable");
    table.innerHTML = "";

    db.collection("kritik").orderBy("waktu","desc").get().then(snap=>{
        let no = 1;
        snap.forEach(doc=>{
            let d = doc.data();
            table.innerHTML += `
            <tr>
                <td>${no++}</td>
                <td>${d.nama}</td>
                <td>${d.jenis}</td>
                <td>${d.pesan}</td>
                <td>${d.rating}</td>
                <td><button class="hapus" onclick="hapusData('${doc.id}')">Hapus</button></td>
            </tr>`;
        });
    });
}

/* HAPUS */
function hapusData(id){
    db.collection("kritik").doc(id).delete().then(tampilkanData);
}

/* LOGIN */
function openLogin(){
    document.getElementById("loginModal").style.display="flex";
}

function loginAdmin(){
    let user = document.getElementById("username").value;
    let pass = document.getElementById("password").value;

    if(user==="12345" && pass==="12345"){
        document.getElementById("adminPanel").style.display="block";
        document.getElementById("loginModal").style.display="none";
        tampilkanData();
    }else{
        alert("Username / Password salah");
    }
}

/* LOAD AWAL */
window.onload = function(){
    tampilkanData();
};
</script>

</body>
</html>
