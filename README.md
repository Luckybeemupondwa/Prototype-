<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Lucky Bee POS</title>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<style>

body{
font-family:Arial;
background:#0f172a;
color:white;
margin:0;
}

header{
background:#facc15;
color:black;
padding:15px;
display:flex;
justify-content:space-between;
}

.container{
max-width:1100px;
margin:auto;
padding:20px;
}

.card{
background:#1e293b;
padding:20px;
border-radius:10px;
margin-bottom:20px;
}

input,select{
padding:8px;
margin:5px;
border:none;
border-radius:6px;
}

button{
padding:7px 12px;
background:#22c55e;
border:none;
border-radius:6px;
color:white;
cursor:pointer;
}

.product{
background:#334155;
padding:10px;
margin-top:6px;
border-radius:6px;
display:flex;
justify-content:space-between;
}

.low{
color:#f87171;
font-weight:bold;
}

.statGrid{
display:grid;
grid-template-columns:repeat(3,1fr);
gap:10px;
}

.stat{
background:#f59e0b;
color:black;
padding:10px;
border-radius:8px;
text-align:center;
}

#receipt{
background:white;
color:black;
padding:15px;
border-radius:6px;
}

footer{
text-align:center;
padding:15px;
background:#020617;
margin-top:20px;
}

</style>
</head>

<body>

<!-- LOGIN -->

<div id="loginPage" style="display:flex;justify-content:center;align-items:center;height:100vh">

<div class="card">

<h2>🐝 Lucky Bee POS</h2>

<input id="user" placeholder="Username"><br>
<input id="pass" type="password" placeholder="Password"><br><br>

<button onclick="login()">Login</button>

<p>admin / 1234</p>
<p>cashier / 1111</p>

</div>

</div>

<!-- APP -->

<div id="app" style="display:none">

<header>

<b>🐝 Lucky Bee POS</b>

<button onclick="logout()">Logout</button>

</header>

<div class="container">

<!-- DASHBOARD -->

<div class="card">

<h3>Dashboard</h3>

<div class="statGrid">

<div class="stat">
<h2 id="money">-</h2>
Money (K)
</div>

<div class="stat">
<h2 id="sales">-</h2>
Items Sold
</div>

<div class="stat">
<h2 id="products">-</h2>
Products
</div>

</div>

</div>

<!-- ADD PRODUCT -->

<div class="card">

<h3>Add Product</h3>

<input id="pname" placeholder="Product name">

<input id="pprice" placeholder="Price (K)">

<input id="pstock" placeholder="Stock">

<button onclick="addProduct()">Add Product</button>

</div>

<!-- SEARCH -->

<div class="card">

<input id="search" placeholder="Search inventory..." onkeyup="render()">

</div>

<!-- INVENTORY -->

<div class="card">

<h3>Inventory</h3>

<div id="inventory"></div>

</div>

<!-- CART -->

<div class="card">

<h3>Cart</h3>

<div id="cart">Cart empty</div>

<button onclick="checkout()">Checkout</button>

</div>

<!-- RECEIPT -->

<div class="card">

<h3>Receipt</h3>

<div id="receipt">No receipt yet</div>

<button onclick="printReceipt()">Print Receipt</button>

</div>

<!-- SALES GRAPH -->

<div class="card">

<h3>Sales Graph</h3>

<canvas id="chart"></canvas>

</div>

<!-- REPORT -->

<div class="card">

<h3>Download Sales Report</h3>

<button onclick="downloadReport()">Download CSV</button>

</div>

</div>

<footer>
© 2026 Lucky Bee POS — All Rights Reserved
</footer>

</div>

<script>

let products=[]
let cart=[]
let sales=[]
let chart

function login(){

let u=document.getElementById("user").value.trim().toLowerCase()
let p=document.getElementById("pass").value.trim()

if((u==="admin" && p==="1234") || (u==="cashier" && p==="1111")){

document.getElementById("loginPage").style.display="none"
document.getElementById("app").style.display="block"

render()

}else{

alert("Wrong login")

}

}

function logout(){

location.reload()

}

function addProduct(){

let name=document.getElementById("pname").value
let price=document.getElementById("pprice").value
let stock=document.getElementById("pstock").value

products.push({

id:Date.now(),
name:name,
price:Number(price),
stock:Number(stock)

})

render()

}

function addCart(id){

let p=products.find(x=>x.id==id)

if(p.stock<=0){

alert("Out of stock")
return

}

let item=cart.find(c=>c.id==id)

if(item){

item.qty++

}else{

cart.push({id:p.id,name:p.name,price:p.price,qty:1})

}

render()

}

function removeCart(id){

cart=cart.filter(c=>c.id!=id)

render()

}

function checkout(){

if(cart.length==0){

alert("Cart empty")
return

}

let receipt="Lucky Bee POS<br><br>"

let total=0

cart.forEach(c=>{

let p=products.find(x=>x.id==c.id)

p.stock-=c.qty

let cost=c.qty*c.price

total+=cost

sales.push(c)

receipt+=c.name+" x"+c.qty+" - K"+cost+"<br>"

})

receipt+="<br>Total: K"+total

document.getElementById("receipt").innerHTML=receipt

cart=[]

render()

updateChart()

}

function render(){

let search=document.getElementById("search").value.toLowerCase()

let inv=""

products
.filter(p=>p.name.toLowerCase().includes(search))
.forEach(p=>{

let warn=p.stock<=5?"<span class='low'>⚠ Low Stock</span>":""

inv+=`

<div class="product">

<div>

${p.name} (K${p.price}) Stock:${p.stock} ${warn}

</div>

<button onclick="addCart(${p.id})">Add</button>

</div>

`

})

document.getElementById("inventory").innerHTML=inv

document.getElementById("cart").innerHTML=

cart.length==0?"Cart empty":

cart.map(c=>`

<div class="product">

${c.name} x${c.qty}

<button onclick="removeCart(${c.id})">Remove</button>

</div>

`).join("")

let money=0
let items=0

sales.forEach(s=>{

money+=s.price*s.qty
items+=s.qty

})

document.getElementById("money").innerText=money||"-"
document.getElementById("sales").innerText=items||"-"
document.getElementById("products").innerText=products.length||"-"

}

function printReceipt(){

let r=document.getElementById("receipt").innerHTML

let w=window.open()

w.document.write(r)

w.print()

}

function updateChart(){

let data={}

sales.forEach(s=>{

data[s.name]=(data[s.name]||0)+s.qty

})

let labels=Object.keys(data)
let values=Object.values(data)

if(chart){chart.destroy()}

chart=new Chart(document.getElementById("chart"),{

type:"bar",

data:{
labels:labels,
datasets:[{
label:"Items Sold",
data:values,
backgroundColor:"#fbbf24"
}]
}

})

}

function downloadReport(){

let csv="Product,Qty\n"

sales.forEach(s=>{

csv+=s.name+","+s.qty+"\n"

})

let blob=new Blob([csv])

let a=document.createElement("a")

a.href=URL.createObjectURL(blob)

a.download="sales_report.csv"

a.click()

}

</script>

</body>
</html>
