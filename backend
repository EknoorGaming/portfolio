const express = require("express");
const http = require("http");
const { Server } = require("socket.io");
const mineflayer = require("mineflayer");

const app = express();
const server = http.createServer(app);
const io = new Server(server, {
    cors: {
        origin: "*", // Allow frontend to connect
    },
});

// Create bot instance
let bot;

function createBot() {
    bot = mineflayer.createBot({
        username: "AFK_Bot",
        host: "pleasebagadomyname.aternos.me",
        port: 45179,
        version: "1.21",
        hideErrors: false
    });

    bot.on("login", () => {
        console.log("✅ Bot has logged in!");
        io.emit("bot_status", "🟢 Bot is online");
        antiAFK();
    });

    bot.on("end", () => {
        console.log("❌ Bot disconnected.");
        io.emit("bot_status", "🔴 Bot is offline");
        setTimeout(createBot, 5000); // Reconnect after 5 seconds
    });

    bot.on("kicked", (reason) => {
        console.log(`⚠️ Bot was kicked: ${reason}`);
        io.emit("bot_status", "🔴 Bot was kicked!");
        if (reason = "Connection throttled! Please wait before reconnecting.") {
            setTimeout(20)
        }
    });

    bot.on("error", (err) => {
        console.log(`❌ Error: ${err}`);
        io.emit("bot_status", "⚠️ Bot encountered an error");
    });

    bot.on("chat", (username, message) => {
        console.log(`${username}: ${message}`);
        io.emit("chat_message", { username, message });
    });

    io.on("connection", (socket) => {
        console.log("A user connected");
        io.emit("bot_status", bot ? "🟢 Bot is online" : "🔴 Bot is offline");

        socket.on("send_chat", (msg) => {
            if (bot) {
                bot.chat(msg);
            }
        });

        socket.on("reconnect_bot", () => {
            console.log("🔄 Reconnecting bot...");
            if (bot) bot.end();
            createBot();
        });
    });
}

// Start bot
createBot();

server.listen(3000, () => {
    console.log("🚀 Server running on http://localhost:3000");
});

// Anti-AFK function
function antiAFK() {
    setInterval(() => {
        if (!bot || !bot.entity) return;

        const actions = [
            () => bot.setControlState("jump", true),  // Jump for a second
            () => bot.setControlState("jump", false),
            () => bot.look(Math.random() * Math.PI, Math.random() * Math.PI), // Move head
            () => bot.setControlState("sneak", true),  // Sneak for a moment
            () => bot.setControlState("sneak", false)
        ];

        const action = actions[Math.floor(Math.random() * actions.length)];
        action();
    }, 30 * 1000); // Every 30 seconds
}
