# TKR-Library
<!DOCTYPE html>
<html>
<head>
    <title>TKR Library Management System</title>
    <style>
        body { font-family: Arial; background: #f2f2f2; margin: 0; }
        .box { width: 90%; max-width: 500px; margin: 20px auto; padding: 20px; background: white; border-radius: 10px; text-align: center; }
        .logo { width: 90px; margin-bottom: 10px; }
        input { width: 100%; padding: 10px; margin: 6px 0; }
        button { width: 100%; padding: 10px; margin-top: 8px; background: #004aad; color: white; border: none; border-radius: 5px; cursor: pointer; }
        button:hover { background: #00337a; }
        .hidden { display: none; }

        /* Table styling */
        .table-wrapper { overflow-x: auto; margin-top: 10px; }
        table { width: 100%; border-collapse: collapse; table-layout: fixed; }
        th, td { border: 1px solid black; padding: 8px; text-align: center; vertical-align: middle; min-height: 40px; word-wrap: break-word; }
        th { background-color: #004aad; color: white; }
        .action-btn { padding: 5px 10px; font-size: 14px; cursor: pointer; background-color: #007bff; border: none; border-radius: 4px; color: white; transition: background-color 0.3s ease; }
        .action-btn:hover { background-color: #0056b3; }

        @media (max-width: 600px) {
            .box { width: 95%; padding: 15px; }
            input, button { font-size: 16px; }
        }
    </style>
</head>
<body>

<div class="box" id="welcomePage">
    <img id="logo1" class="logo" alt="College Logo">
    <h2>Welcome to TKR Library</h2>
    <button onclick="goLogin()">Enter</button>
</div>

<div class="box hidden" id="loginPage">
    <img id="logo2" class="logo" alt="College Logo">
    <h3>Login</h3>
    <input type="text" id="username" placeholder="Username">
    <input type="password" id="password" placeholder="Password">
    <label><input type="checkbox" onclick="togglePassword()"> Show Password</label>
    <button onclick="login()">Login</button>
</div>

<div class="box hidden" id="homePage">
    <img id="logo3" class="logo" alt="College Logo">
    <h3 id="branchDisplay"></h3>
    <input type="text" id="bookName" placeholder="Book Name">
    <input type="text" id="authorName" placeholder="Author Name">
    <button onclick="searchBook()">Search</button>
    <button onclick="viewBranchBooks()">View My Branch Books</button>
    <button id="adminBtn" onclick="openAdmin()" style="display:none;">Admin Panel</button>
    <button onclick="viewDashboard()">Dashboard</button>
    <button id="myIssuedBtn" onclick="viewMyIssuedBooks()" style="display:none;">My Issued Books</button>
    <button onclick="logout()">Logout</button>
</div>

<div class="box hidden" id="resultPage">
    <img id="logo4" class="logo" alt="College Logo">
    <div class="table-wrapper">
        <table id="resultTable"></table>
    </div>
    <button onclick="goBack()">Back</button>
</div>

<div class="box hidden" id="adminPage">
    <img id="logo5" class="logo" alt="College Logo">
    <h3>Admin Panel</h3>
    <input type="text" id="newName" placeholder="Book Name">
    <input type="text" id="newAuthor" placeholder="Author">
    <input type="text" id="newBranch" placeholder="Branch (EEE/CSD/CSE/CSM/CIVIL)">
    <input type="number" id="newCopies" placeholder="Copies">
    <button onclick="addBook()">Add Book</button>
    <h4>Upload Logo</h4>
    <input type="file" id="logoUpload" accept="image/*">
    <button onclick="uploadLogo()">Upload</button>

    <h4>All Books</h4>
    <div class="table-wrapper">
        <table id="adminTable"></table>
    </div>

    <h4>Issued Books</h4>
    <div class="table-wrapper">
        <table id="issuedTable"></table>
    </div>
    <button onclick="goHome()">Back</button>
</div>

<div class="box hidden" id="dashboardPage">
    <img id="logo6" class="logo" alt="College Logo">
    <h3>Library Dashboard</h3>
    <p id="totalBooks"></p>
    <p id="branchBooks"></p>
    <p id="totalIssued"></p>
    <button onclick="goHomeFromDashboard()">Back</button>
</div>

<div class="box hidden" id="studentIssuedPage">
    <img id="logo7" class="logo" alt="College Logo">
    <h3>My Issued Books</h3>
    <div class="table-wrapper">
        <table id="myIssuedTable"></table>
    </div>
    <button onclick="backToHome()">Back</button>
</div>

<script>
let defaultLogo = "https://upload.wikimedia.org/wikipedia/commons/1/1b/College_logo_sample.png";

let books = [
    { name: "Electrical Engineering", author: "V.K. Mehta", branch: "EEE", copies: 10 },
    { name: "Power Systems", author: "I.J. Nagrath", branch: "EEE", copies: 6 },
    { name: "Machine Learning", author: "Tom Mitchell", branch: "CSD", copies: 8 },
    { name: "Artificial Intelligence", author: "Stuart Russell", branch: "CSD", copies: 5 },
    { name: "Data Structures", author: "Schaum Series", branch: "CSE", copies: 12 },
    { name: "Operating Systems", author: "Galvin", branch: "CSE", copies: 8 },
    { name: "Robotics", author: "John Craig", branch: "CSM", copies: 5 },
    { name: "Concrete Technology", author: "M.S. Shetty", branch: "CIVIL", copies: 7 },
    { name: "Digital Electronics", author: "Boylestad", branch: "EEE", copies: 7 },
    { name: "Signals & Systems", author: "Oppenheim", branch: "EEE", copies: 5 },
    { name: "Database Systems", author: "Ramakrishnan", branch: "CSE", copies: 9 },
    { name: "Microprocessors", author: "Gaonkar", branch: "CSM", copies: 6 }
];

let issuedBooks = []; 
let currentUser = "";
let userBranch = "";

// Default logo unless changed
window.onload = function() {
    let logo = localStorage.getItem("logo") || defaultLogo;
    for(let i = 1; i <= 7; i++) {
        let img = document.getElementById("logo"+i);
        if(img) img.src = logo;
    }
};

function goLogin(){ welcomePage.classList.add("hidden"); loginPage.classList.remove("hidden"); }
function togglePassword(){ password.type = password.type==="password" ? "text" : "password"; }

function login(){
    let u = username.value.trim(), p = password.value.trim();
    if(u==="ashketchum" && p==="Ash@2204"){ currentUser="admin"; adminBtn.style.display="block"; branchDisplay.innerText="Admin Access"; myIssuedBtn.style.display="none"; }
    else if(/^25K91A\d{4}$/.test(u) && u===p){ currentUser="student"; 
        let num=parseInt(u.slice(-4));
        let ranges=[{name:"EEE",min:1,max:300},{name:"CSD",min:301,max:499},{name:"CSE",min:500,max:700},{name:"CSM",min:701,max:900},{name:"CIVIL",min:901,max:1000}];
        let found=ranges.find(r=>num>=r.min&&num<=r.max);
        if(!found){ alert("Invalid Roll Number"); return; }
        userBranch=found.name; branchDisplay.innerText=`Welcome ${u} (${userBranch})`; adminBtn.style.display="none"; myIssuedBtn.style.display="block";
    } else { alert("Invalid Login!"); return; }
    loginPage.classList.add("hidden"); homePage.classList.remove("hidden");
}

function logout(){ currentUser=""; userBranch=""; adminBtn.style.display="none"; myIssuedBtn.style.display="none"; homePage.classList.add("hidden"); loginPage.classList.remove("hidden"); username.value=""; password.value=""; bookName.value=""; authorName.value=""; }

function searchBook(){ let b=bookName.value.toLowerCase(), a=authorName.value.toLowerCase(); let filteredBooks=books.filter(x=>(currentUser==="admin"||x.branch===userBranch)&&(!b||x.name.toLowerCase().includes(b))&&(!a||x.author.toLowerCase().includes(a))); showResults(filteredBooks); }
function viewBranchBooks(){ let filteredBooks=currentUser==="admin"?books:books.filter(b=>b.branch===userBranch); showResults(filteredBooks); }
function showResults(res){ resultTable.innerHTML="<tr><th>Book</th><th>Author</th><th>Branch</th><th>Available</th><th>Copies</th><th>Action</th></tr>"; if(res.length){ res.forEach(x=>{ let action=(currentUser==="student"&&x.branch===userBranch&&x.copies>0)?`<button class="action-btn" onclick="issueBook('${x.name}')">Issue</button>`:""; resultTable.innerHTML+=`<tr><td>${x.name}</td><td>${x.author}</td><td>${x.branch}</td><td>${x.copies>0?"Yes":"No"}</td><td>${x.copies}</td><td>${action}</td></tr>`; }); homePage.classList.add("hidden"); resultPage.classList.remove("hidden"); } else{ alert("No books found!"); } }
function goBack(){ resultPage.classList.add("hidden"); homePage.classList.remove("hidden"); }

function openAdmin(){ if(currentUser!=="admin"){ alert("Access Denied!"); return; } homePage.classList.add("hidden"); adminPage.classList.remove("hidden"); loadBooks(); loadIssued(); }
function goHome(){ adminPage.classList.add("hidden"); homePage.classList.remove("hidden"); }

function addBook(){ if(currentUser!=="admin"){ alert("Only admin can add books."); return; } if(!newName.value||!newAuthor.value||!newBranch.value||newCopies.value<=0){ alert("Fill all fields"); return; } books.push({name:newName.value,author:newAuthor.value,branch:newBranch.value.toUpperCase(),copies:parseInt(newCopies.value)}); alert("Book added!"); loadBooks(); newName.value=""; newAuthor.value=""; newBranch.value=""; newCopies.value=""; }

function loadBooks(){ adminTable.innerHTML="<tr><th>Book</th><th>Author</th><th>Branch</th><th>Copies</th></tr>"; books.forEach(b=>{ adminTable.innerHTML+=`<tr><td>${b.name}</td><td>${b.author}</td><td>${b.branch}</td><td>${b.copies}</td></tr>`; }); }

function issueBook(bookName){ let book=books.find(x=>x.name===bookName&&x.branch===userBranch); if(!book||book.copies===0){ alert("Book not available"); return; } book.copies--; let due=new Date(); due.setDate(due.getDate()+15); let dueDateStr=due.toISOString().split('T')[0]; issuedBooks.push({book:book.name,branch:book.branch,studentRoll:username.value,dueDate:dueDateStr}); alert(`Book issued! Due: ${dueDateStr}`); refreshAllTables(); }
function returnBook(bookName){ let index=issuedBooks.findIndex(x=>x.book===bookName&&x.studentRoll===username.value); if(index===-1){ alert("No record"); return; } let book=books.find(x=>x.name===bookName&&x.branch===userBranch); if(book) book.copies++; issuedBooks.splice(index,1); alert("Book returned"); refreshAllTables(); }

function loadIssued(){ issuedTable.innerHTML="<tr><th>Book</th><th>Branch</th><th>Issued To</th><th>Due Date</th><th>Action</th></tr>"; let today=new Date().toISOString().split('T')[0]; issuedBooks.forEach(x=>{ let overdue=x.dueDate<today?"style='background-color:#f8d7da'":""; issuedTable.innerHTML+=`<tr ${overdue}><td>${x.book}</td><td>${x.branch}</td><td>${x.studentRoll}</td><td>${x.dueDate}</td><td>${currentUser==="admin"?`<button class="action-btn" onclick="returnBook('${x.book}')">Return</button>`:""}</td></tr>`; }); }
function loadMyIssuedBooks(){ myIssuedTable.innerHTML="<tr><th>Book</th><th>Branch</th><th>Due Date</th><th>Action</th></tr>"; let myBooks=issuedBooks.filter(x=>x.studentRoll===username.value); if(myBooks.length===0){ myIssuedTable.innerHTML+=`<tr><td colspan="4">No issued books</td></tr>`; return; } let today=new Date().toISOString().split('T')[0]; myBooks.forEach(x=>{ let overdue=x.dueDate<today?"style='background-color:#f8d7da'":""; myIssuedTable.innerHTML+=`<tr ${overdue}><td>${x.book}</td><td>${x.branch}</td><td>${x.dueDate}</td><td><button class="action-btn" onclick="returnBook('${x.book}')">Return</button></td></tr>`; }); }
function refreshAllTables(){ if(currentUser==="admin") loadIssued(); if(currentUser==="student") loadMyIssuedBooks(); if(!resultPage.classList.contains("hidden")) searchBook(); }

function viewDashboard(){ homePage.classList.add("hidden"); dashboardPage.classList.remove("hidden"); totalBooks.innerText="Total Books: "+books.reduce((acc,b)=>acc+b.copies,0); let branchCounts=books.reduce((acc,b)=>{ acc[b.branch]=(acc[b.branch]||0)+b.copies; return acc; },{}); branchBooks.innerText="Books per Branch: "+JSON.stringify(branchCounts); totalIssued.innerText="Total Issued: "+issuedBooks.length; }
function goHomeFromDashboard(){ dashboardPage.classList.add("hidden"); homePage.classList.remove("hidden"); }

function viewMyIssuedBooks(){ homePage.classList.add("hidden"); studentIssuedPage.classList.remove("hidden"); loadMyIssuedBooks(); }
function backToHome(){ studentIssuedPage.classList.add("hidden"); homePage.classList.remove("hidden"); }

function uploadLogo(){ let file=logoUpload.files[0]; if(!file){ alert("Select an image"); return; } let reader=new FileReader(); reader.onload=e=>{ localStorage.setItem("logo",e.target.result); location.reload(); }; reader.readAsDataURL(file); }
</script>

</body>
</html>
