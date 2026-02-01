let users = JSON.parse(localStorage.getItem("users")) || [];
let currentUser = JSON.parse(localStorage.getItem("currentUser"));
let investorMode = false;
let currentOTP = null;

function now(){return new Date().toLocaleString();}

// SIGNUP
function signup() {
  const user = {
    username: username.value,
    email: email.value,
    password: password.value,
    pin: pin.value,
    balance: 5000,
    savings: 1000,
    cards: [],
    crypto:{Bitcoin:0,Ethereum:0,USDT:0},
    transactions: [],
    kyc:"Pending (Simulation)",
    tier:"Basic",
    restricted:false,
    emailAlerts:[]
  };
  users.push(user);
  localStorage.setItem("users", JSON.stringify(users));
  alert("Account created");
  location.href="index.html";
}

// LOGIN - REQUEST OTP
function sendOTP(){
  currentOTP = Math.floor(100000 + Math.random()*900000);
  alert("Demo OTP: " + currentOTP); // Simulated code
}

// LOGIN - VERIFY OTP + PIN
function verifyOTP(){
  currentUser = users.find(u =>
    u.username === loginUser.value &&
    u.password === loginPass.value &&
    u.pin === loginPin.value
  );
  if(!currentUser) return alert("Invalid credentials");
  if(Number(otpInput.value) !== currentOTP) return alert("Invalid verification code");
  localStorage.setItem("currentUser", JSON.stringify(currentUser));
  location.href="dashboard.html";
}

// DASHBOARD
function loadDashboard(){
  currentUser = JSON.parse(localStorage.getItem("currentUser"));
  if(!currentUser) return location.href="index.html";
  balance.innerText = currentUser.balance;
  kycDisplay.innerText = currentUser.kyc;
  tierDisplay.innerText = currentUser.tier;
  showChart();
  showEmailAlerts();
}

// SIMULATE KYC UPLOAD
function simulateUpload(){
  document.getElementById("kycStatus").innerText = "KYC Status: Verified (Simulation)";
  alert("Sample document accepted (Simulation)");
}

// INVESTOR MODE
function toggleInvestorMode(){
  investorMode = document.getElementById("investorMode").checked;
  investorMsg.innerText = investorMode?"Investor Mode ON":"Investor Mode OFF";
  if(investorMode){
    currentUser.tier="Pro (Investor Demo)";
    currentUser.kyc="Verified (Simulation)";
    saveAndRefresh();
  }
}

// TRANSFER
function transfer(){
  let amt = Number(amount.value);
  if(amt<=0 || amt>currentUser.balance) return alert("Invalid amount");
  let receiverUser = users.find(u=>u.username===receiver.value);
  if(!receiverUser) return alert("Receiver not found");
  currentUser.balance-=amt;
  receiverUser.balance+=amt;

  let msgOut=`Transfer -$${amt} to ${receiver.value} (${now()})`;
  let msgIn=`Transfer +$${amt} from ${currentUser.username} (${now()})`;

  currentUser.transactions.push(msgOut);
  currentUser.emailAlerts.push("Outgoing: "+msgOut);
  receiverUser.transactions.push(msgIn);
  receiverUser.emailAlerts.push("Incoming: "+msgIn);

  users = users.map(u=>u.username===receiverUser.username?receiverUser:u);
  localStorage.setItem("users", JSON.stringify(users));
  saveAndRefresh();
  fakeEmail("Transfer sent to "+receiver.value);
}

// BANK TRANSFER
function bankTransfer(){
  let amt=1000;
  currentUser.balance-=amt;
  currentUser.transactions.push(`Bank Transfer -$${amt} (${now()})`);
  currentUser.emailAlerts.push("Outgoing: Bank Transfer -$"+amt);
  saveAndRefresh();
  fakeEmail("Bank transfer processed");
}

// ADD CARD
function addCard(){
  let cardNum="**** **** **** 1234";
  currentUser.cards.push(cardNum);
  currentUser.transactions.push(`Card Added ${cardNum} (${now()})`);
  currentUser.emailAlerts.push("Card added: "+cardNum);
  saveAndRefresh();
}

// CRYPTO
function sendCrypto(){
  currentUser.crypto.Bitcoin+=0.01;
  currentUser.transactions.push(`Bitcoin +0.01 (${now()})`);
  currentUser.emailAlerts.push("Crypto received: Bitcoin +0.01");
  saveAndRefresh();
}

// GIFT CARD
function redeemGift(){
  currentUser.balance+=200;
  currentUser.transactions.push(`Gift Card +$200 (${now()})`);
  currentUser.emailAlerts.push("Gift Card Redeemed +$200");
  saveAndRefresh();
}

// ARTIST PAYOUT
function payoutArtist(){
  let amt=Number(artistAmount.value);
  if(amt<=0 || amt>currentUser.balance) return alert("Invalid amount");
  currentUser.balance-=amt;
  currentUser.transactions.push(`Payout to ${artistName.value} -$${amt} (${now()})`);
  currentUser.emailAlerts.push(`Outgoing: Artist payout to ${artistName.value} -$${amt}`);
  saveAndRefresh();
}

// SHOW EMAIL ALERTS
function showEmailAlerts(){
  emailAlerts.innerHTML="";
  currentUser.emailAlerts.forEach(msg=>{
    let li=document.createElement("li");
    li.innerText=msg;
    emailAlerts.appendChild(li);
  });
}

// CHART
function showChart(){
  const ctx=document.getElementById('balanceChart');
  if(!ctx) return;
  new Chart(ctx,{
    type:'bar',
    data:{
      labels:['Balance','Savings'],
      datasets:[{label:'USD',data:[currentUser.balance,currentUser.savings],backgroundColor:['#ff6f00','#1b4dff']}]
    }
  });
}

// LOGOUT
function logout(){
  localStorage.removeItem("currentUser");
  location.href="index.html";
}

// HELPERS
function fakeEmail(msg){
  alert("📧 Demo Alert: "+msg+(investorMode?" [Investor Mode]":""));
}
function saveAndRefresh(){
  localStorage.setItem("currentUser",JSON.stringify(currentUser));
  loadDashboard();
}
