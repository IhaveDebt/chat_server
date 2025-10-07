/**
 * Secure Chat Server (chat_server.ts)
 *
 * A WebSocket-based chat server that includes message hashing for integrity.
 *
 * Run:
 *   npm install ws
 *   ts-node src/chat_server.ts
 */

import WebSocket, { WebSocketServer } from 'ws';
import crypto from 'crypto';

class SecureChatServer {
  private wss: WebSocketServer;

  constructor(port: number) {
    this.wss = new WebSocketServer({ port });
    this.wss.on('connection', ws => {
      ws.on('message', data => {
        try {
          const text = data.toString();
          this.broadcast(ws, text);
        } catch (err) {
          console.error('Invalid message', err);
        }
      });
    });
    console.log(`SecureChatServer running on port ${port}`);
  }

  private broadcast(sender: WebSocket, msg: string) {
    // compute a lightweight integrity signature (sha256 hex)
    const signature = crypto.createHash('sha256').update(msg).digest('hex');
    const packet = JSON.stringify({ msg, signature, ts: Date.now() });
    for (const client of this.wss.clients) {
      if (client !== sender && client.readyState === WebSocket.OPEN) {
        client.send(packet);
      }
    }
  }
}

if (require.main === module) {
  const PORT = parseInt(process.env.PORT || '8080', 10);
  new SecureChatServer(PORT);
}
