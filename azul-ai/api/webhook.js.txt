export default async function handler(req, res) {
  if (req.method === "GET") {
    if (req.query["hub.verify_token"] === process.env.IG_VERIFY_TOKEN) {
      return res.send(req.query["hub.challenge"]);
    } else {
      return res.status(403).send("Invalid token");
    }
  }

  if (req.method === "POST") {
    const body = req.body;
    console.log("Webhook event:", JSON.stringify(body, null, 2));

    if (body.entry?.[0]?.messaging?.[0]?.message) {
      const msg = body.entry[0].messaging[0].message.text;
      const sender = body.entry[0].messaging[0].sender.id;

      if (/bot|qué sos|who are you/i.test(msg)) {
        await sendIGMessage(sender, "💙 I’m Azul — an AI-generated artist persona. For official stuff, a human will reply.");
        return res.status(200).send("ok");
      }

      await sendIGMessage(sender, "Hey! Azul here 👋 thanks for your message.");
    }

    return res.status(200).send("EVENT_RECEIVED");
  }
}

async function sendIGMessage(to, text) {
  await fetch(`https://graph.facebook.com/v19.0/me/messages?access_token=${process.env.IG_ACCESS_TOKEN}`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      messaging_type: "RESPONSE",
      recipient: { id: to },
      message: { text }
    })
  });
}
