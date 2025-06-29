function pollInbox() {
  setInterval(async () => {
    const inboxRes = await fetch("https://api.mail.tm/messages", {
      headers: { Authorization: `Bearer ${emailData.token}` }
    });

    const inbox = await inboxRes.json();
    const messages = inbox["hydra:member"];

    const list = document.getElementById("messageList");
    list.innerHTML = "";

    if (messages.length === 0) {
      list.innerHTML = "<li>No new messages yet...</li>";
      return;
    }

    for (const msg of messages) {
      // Fetch full message details
      const msgRes = await fetch(`https://api.mail.tm/messages/${msg.id}`, {
        headers: { Authorization: `Bearer ${emailData.token}` }
      });
      const fullMsg = await msgRes.json();

      const li = document.createElement("li");
      li.innerHTML = `
        <strong>From:</strong> ${fullMsg.from.name || fullMsg.from.address} <br>
        <strong>Email:</strong> ${fullMsg.from.address} <br>
        <strong>Subject:</strong> ${fullMsg.subject} <br>
        <strong>Preview:</strong> ${fullMsg.intro || 'No preview available.'}
      `;
      list.appendChild(li);
    }
  }, 10000); // every 10 seconds
}
