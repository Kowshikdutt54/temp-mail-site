<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>TempMailPro - Free Temporary Email</title>
  <link rel="stylesheet" href="style.css"/>
</head>
<body>
  <div class="container">
    <h1>📬 TempMailPro</h1>
    <p>Get a free temporary email address. Stay private, stay spam-free.</p>
    
    <div class="email-box">
      <button id="generateBtn">🎲 Generate New Email</button>
      <input type="text" id="emailAddress" readonly />
      <button onclick="copyEmail()">📋 Copy</button>
    </div>

    <div class="inbox">
      <h2>📥 Inbox</h2>
      <ul id="messageList">
        <li>Waiting for messages...</li>
      </ul>
    </div>
  </div>

  <script src="script.js"></script>
</body>
</html>
body {
  font-family: 'Segoe UI', sans-serif;
  background: #f3f4f6;
  margin: 0;
  padding: 20px;
}

.container {
  max-width: 600px;
  margin: auto;
  background: white;
  border-radius: 12px;
  padding: 25px;
  box-shadow: 0 0 12px rgba(0,0,0,0.1);
  text-align: center;
}

h1 {
  margin-bottom: 10px;
}

.email-box {
  margin: 20px 0;
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
  justify-content: center;
}

#emailAddress {
  width: 100%;
  max-width: 300px;
  padding: 10px;
  font-size: 16px;
  border: 2px solid #ddd;
  border-radius: 6px;
  text-align: center;
}

button {
  background-color: #2563eb;
  color: white;
  padding: 10px 15px;
  font-size: 15px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
}

button:hover {
  background-color: #1e40af;
}

.inbox {
  text-align: left;
  margin-top: 30px;
}

#messageList {
  list-style: none;
  padding: 0;
}
#messageList li {
  background: #f9fafb;
  padding: 10px;
  border-bottom: 1px solid #eee;
  border-radius: 5px;
  margin-bottom: 10px;
}
let emailData = { address: "", id: "", token: "" };

document.getElementById("generateBtn").addEventListener("click", async () => {
  const res = await fetch("https://api.mail.tm/accounts", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      address: `user${Math.floor(Math.random() * 99999)}@mail.tm`,
      password: "tempmail123"
    })
  });

  const data = await res.json();
  emailData.address = data.address;
  emailData.id = data.id;

  const tokenRes = await fetch("https://api.mail.tm/token", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      address: data.address,
      password: "tempmail123"
    })
  });

  const tokenData = await tokenRes.json();
  emailData.token = tokenData.token;

  document.getElementById("emailAddress").value = emailData.address;
  document.getElementById("messageList").innerHTML = "<li>Waiting for messages...</li>";

  pollInbox();
});

function copyEmail() {
  const input = document.getElementById("emailAddress");
  navigator.clipboard.writeText(input.value);
  alert("Copied: " + input.value);
}

function pollInbox() {
  const interval = setInterval(async () => {
    const inboxRes = await fetch("https://api.mail.tm/messages", {
      headers: { Authorization: `Bearer ${emailData.token}` }
    });
    const inbox = await inboxRes.json();

    if (inbox["hydra:member"].length > 0) {
      const list = document.getElementById("messageList");
      list.innerHTML = "";
      inbox["hydra:member"].forEach((msg) => {
        const li = document.createElement("li");
        li.innerHTML = `<strong>${msg.from.address}</strong><br>${msg.subject}`;
        list.appendChild(li);
      });
    }
  }, 10000); // every 10 seconds
}
