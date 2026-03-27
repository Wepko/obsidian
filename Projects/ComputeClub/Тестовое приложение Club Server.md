Отлично! Давай создадим тестовый скрипт Club Server с заглушками. Он будет эмулировать всю логику, но без реального взаимодействия с ПК.

---

## 📁 Создаем тестовый Club Server

### Структура проекта

```
club-server-test/
├── package.json
├── .env.example
├── club-server.js
└── README.md
```

---

## 📦 package.json

```json
{
  "name": "club-server-test",
  "version": "1.0.0",
  "description": "Test Club Server for ESports Arena",
  "main": "club-server.js",
  "scripts": {
    "start": "node club-server.js",
    "dev": "nodemon club-server.js"
  },
  "dependencies": {
    "fastify": "^4.26.0",
    "ws": "^8.14.0",
    "dotenv": "^16.3.0"
  },
  "devDependencies": {
    "nodemon": "^3.0.0"
  }
}
```

---

## 🚀 club-server.js (полный код с заглушками)

```javascript
#!/usr/bin/env node

/**
 * ESports Arena - Club Server (Test Version)
 * 
 * Эмуляция работы Club Server с заглушками
 * HTTP порт: 3000 (админка)
 * WebSocket порт: 3001 (для ПК)
 */

const fastify = require('fastify')({ logger: true });
const WebSocket = require('ws');
const dotenv = require('dotenv');

dotenv.config();

// ============================================
// 1. НАСТРОЙКА И ХРАНИЛИЩА
// ============================================

const app = fastify;

// Хранилище подключенных ПК (эмуляция)
const pcs = new Map();

// Хранилище активных сессий
const activeSessions = new Map();

// Статистика
const stats = {
    startedAt: new Date(),
    totalConnections: 0,
    totalSessions: 0,
    totalBalance: 0
};

// Конфигурация из .env
const config = {
    clubId: process.env.CLUB_ID || 1,
    clubName: process.env.CLUB_NAME || 'Test Club',
    cloudApiUrl: process.env.CLOUD_API_URL || 'https://api.example.com',
    httpPort: process.env.HTTP_PORT || 3000,
    wsPort: process.env.WS_PORT || 3001
};

// ============================================
// 2. HTTP API (FASTIFY) - ЗАГЛУШКИ
// ============================================

// Главная страница
app.get('/', async (request, reply) => {
    return {
        name: 'ESports Arena Club Server',
        version: '1.0.0',
        status: 'online',
        club: {
            id: config.clubId,
            name: config.clubName
        },
        stats: {
            uptime: Math.floor((Date.now() - stats.startedAt) / 1000),
            pcsConnected: pcs.size,
            activeSessions: activeSessions.size,
            totalSessions: stats.totalSessions,
            totalBalance: stats.totalBalance
        },
        endpoints: {
            health: 'GET /health',
            status: 'GET /status',
            pcs: 'GET /pcs',
            sessions: 'GET /sessions',
            clients: 'GET /clients',
            'start-session': 'POST /start-session',
            'end-session': 'POST /end-session',
            'topup-balance': 'POST /topup'
        }
    };
});

// Проверка здоровья
app.get('/health', async (request, reply) => {
    return {
        status: 'healthy',
        timestamp: new Date().toISOString(),
        uptime: Math.floor((Date.now() - stats.startedAt) / 1000),
        services: {
            http: 'running',
            websocket: 'running',
            cache: 'running'
        }
    };
});

// Детальный статус
app.get('/status', async (request, reply) => {
    const pcsList = Array.from(pcs.values()).map(pc => ({
        id: pc.id,
        ip: pc.ip,
        status: pc.status,
        session: pc.session ? {
            clientId: pc.session.clientId,
            clientName: pc.session.clientName,
            balance: pc.session.balance,
            startedAt: pc.session.startedAt
        } : null
    }));

    return {
        club: {
            id: config.clubId,
            name: config.clubName,
            online: true
        },
        stats: {
            pcs: {
                total: pcs.size,
                online: Array.from(pcs.values()).filter(p => p.status === 'online').length,
                inGame: Array.from(pcs.values()).filter(p => p.session).length
            },
            sessions: {
                active: activeSessions.size,
                total: stats.totalSessions
            },
            finances: {
                totalBalance: stats.totalBalance,
                todayRevenue: 12500 // заглушка
            }
        },
        pcs: pcsList,
        sessions: Array.from(activeSessions.values())
    };
});

// Список ПК
app.get('/pcs', async (request, reply) => {
    const pcsList = Array.from(pcs.values()).map(pc => ({
        id: pc.id,
        name: `PC-${pc.id}`,
        ip: pc.ip,
        status: pc.status,
        isOccupied: !!pc.session,
        sessionId: pc.session?.id || null
    }));

    return {
        total: pcsList.length,
        online: pcsList.filter(p => p.status === 'online').length,
        occupied: pcsList.filter(p => p.isOccupied).length,
        list: pcsList
    };
});

// Активные сессии
app.get('/sessions', async (request, reply) => {
    const sessions = Array.from(activeSessions.values()).map(session => ({
        id: session.id,
        computerId: session.computerId,
        computerName: `PC-${session.computerId}`,
        clientId: session.clientId,
        clientName: session.clientName,
        balance: session.balance,
        startedAt: session.startedAt,
        duration: Math.floor((Date.now() - new Date(session.startedAt).getTime()) / 1000)
    }));

    return {
        active: sessions.length,
        totalToday: stats.totalSessions,
        sessions: sessions
    };
});

// Список клиентов (заглушка)
app.get('/clients', async (request, reply) => {
    return {
        clients: [
            { id: 1, name: 'Иван Петров', phone: '+7 999 123-45-67', balance: 1250, visits: 12 },
            { id: 2, name: 'Алексей Смирнов', phone: '+7 999 234-56-78', balance: 3400, visits: 25 },
            { id: 3, name: 'Дмитрий Иванов', phone: '+7 999 345-67-89', balance: 500, visits: 3 },
            { id: 4, name: 'Сергей Козлов', phone: '+7 999 456-78-90', balance: 890, visits: 7 },
            { id: 5, name: 'Анна Соколова', phone: '+7 999 567-89-01', balance: 2100, visits: 18 }
        ]
    };
});

// Старт сессии (заглушка)
app.post('/start-session', async (request, reply) => {
    const { computerId, clientId } = request.body;
    
    if (!computerId || !clientId) {
        return reply.status(400).send({ error: 'computerId and clientId required' });
    }
    
    const pc = pcs.get(computerId);
    if (!pc) {
        return reply.status(404).send({ error: `PC ${computerId} not found` });
    }
    
    if (pc.session) {
        return reply.status(409).send({ error: 'PC already has active session' });
    }
    
    // Создаем сессию
    const sessionId = Date.now();
    const session = {
        id: sessionId,
        computerId: computerId,
        clientId: clientId,
        clientName: `Клиент ${clientId}`,
        balance: 1000,
        startedAt: new Date().toISOString(),
        tariff: {
            id: 1,
            name: 'Стандарт',
            pricePerHour: 100
        }
    };
    
    activeSessions.set(sessionId, session);
    pc.session = session;
    stats.totalSessions++;
    stats.totalBalance += 1000;
    
    // Эмуляция отправки команды на ПК
    console.log(`[EMULATION] Starting session on PC-${computerId} for client ${clientId}`);
    
    return {
        success: true,
        session: session,
        message: `Session started on PC-${computerId}`
    };
});

// Завершить сессию
app.post('/end-session', async (request, reply) => {
    const { computerId } = request.body;
    
    const pc = pcs.get(computerId);
    if (!pc || !pc.session) {
        return reply.status(404).send({ error: 'No active session on this PC' });
    }
    
    const session = pc.session;
    const duration = Math.floor((Date.now() - new Date(session.startedAt).getTime()) / 1000);
    const cost = (duration / 3600) * session.tariff.pricePerHour;
    
    activeSessions.delete(session.id);
    pc.session = null;
    
    console.log(`[EMULATION] Session ended on PC-${computerId}, duration: ${duration}s, cost: ${cost.toFixed(2)} руб`);
    
    return {
        success: true,
        session: {
            id: session.id,
            duration: duration,
            cost: cost.toFixed(2),
            finalBalance: session.balance - cost
        }
    };
});

// Пополнение баланса
app.post('/topup', async (request, reply) => {
    const { clientId, amount } = request.body;
    
    if (!clientId || !amount) {
        return reply.status(400).send({ error: 'clientId and amount required' });
    }
    
    stats.totalBalance += amount;
    
    // Если клиент сейчас играет на каком-то ПК, обновляем баланс
    for (const [id, session] of activeSessions) {
        if (session.clientId === clientId) {
            session.balance += amount;
            
            // Эмуляция отправки обновления баланса на ПК
            console.log(`[EMULATION] Balance updated for client ${clientId} on PC-${session.computerId}: +${amount} руб`);
        }
    }
    
    return {
        success: true,
        clientId: clientId,
        newBalance: 1000 + amount, // заглушка
        amount: amount,
        message: `Balance topped up by ${amount} rub`
    };
});

// Синхронизация с Cloud (заглушка)
app.post('/sync', async (request, reply) => {
    const data = request.body;
    
    console.log(`[EMULATION] Syncing with cloud:`, data);
    
    return {
        success: true,
        synced: true,
        timestamp: new Date().toISOString()
    };
});

// Логи
app.get('/logs', async (request, reply) => {
    return {
        logs: [
            { timestamp: new Date().toISOString(), level: 'info', message: 'Server started' },
            { timestamp: new Date().toISOString(), level: 'info', message: 'WebSocket server listening on port 3001' },
            { timestamp: new Date().toISOString(), level: 'info', message: 'PC-1 connected' },
            { timestamp: new Date().toISOString(), level: 'info', message: 'Session started on PC-1' }
        ]
    };
});

// ============================================
// 3. WEBSOCKET СЕРВЕР ДЛЯ ПК
// ============================================

const wss = new WebSocket.Server({ port: config.wsPort });

// Эмуляция ПК (для тестирования)
let emulatedPCs = [];

wss.on('connection', (ws, req) => {
    const ip = req.socket.remoteAddress;
    let computerId = null;
    
    console.log(`[WebSocket] New connection from ${ip}`);
    stats.totalConnections++;
    
    ws.on('message', (data) => {
        try {
            const message = JSON.parse(data);
            console.log(`[WebSocket] Received:`, message);
            
            switch (message.type) {
                case 'register':
                    computerId = message.computerId;
                    const pc = {
                        id: computerId,
                        ip: ip,
                        ws: ws,
                        status: 'online',
                        session: null,
                        registeredAt: new Date()
                    };
                    pcs.set(computerId, pc);
                    
                    ws.send(JSON.stringify({
                        type: 'registered',
                        status: 'ok',
                        message: `PC-${computerId} registered successfully`,
                        club: {
                            id: config.clubId,
                            name: config.clubName
                        }
                    }));
                    
                    console.log(`[WebSocket] PC-${computerId} registered from ${ip}`);
                    break;
                    
                case 'start_session':
                    if (computerId) {
                        const pc = pcs.get(computerId);
                        if (pc) {
                            const sessionId = Date.now();
                            const session = {
                                id: sessionId,
                                clientId: message.clientId,
                                clientName: message.clientName || `Client ${message.clientId}`,
                                balance: message.balance || 1000,
                                startedAt: new Date().toISOString()
                            };
                            pc.session = session;
                            activeSessions.set(sessionId, session);
                            
                            ws.send(JSON.stringify({
                                type: 'session_started',
                                status: 'ok',
                                session: session
                            }));
                            
                            console.log(`[WebSocket] Session started on PC-${computerId}`);
                        }
                    }
                    break;
                    
                case 'end_session':
                    if (computerId) {
                        const pc = pcs.get(computerId);
                        if (pc && pc.session) {
                            activeSessions.delete(pc.session.id);
                            pc.session = null;
                            
                            ws.send(JSON.stringify({
                                type: 'session_ended',
                                status: 'ok'
                            }));
                            
                            console.log(`[WebSocket] Session ended on PC-${computerId}`);
                        }
                    }
                    break;
                    
                case 'heartbeat':
                    ws.send(JSON.stringify({ type: 'heartbeat_ack', timestamp: Date.now() }));
                    break;
                    
                default:
                    ws.send(JSON.stringify({
                        type: 'error',
                        message: `Unknown message type: ${message.type}`
                    }));
            }
        } catch (err) {
            console.error('[WebSocket] Error parsing message:', err);
            ws.send(JSON.stringify({
                type: 'error',
                message: 'Invalid JSON'
            }));
        }
    });
    
    ws.on('close', () => {
        if (computerId) {
            const pc = pcs.get(computerId);
            if (pc && pc.session) {
                activeSessions.delete(pc.session.id);
            }
            pcs.delete(computerId);
            console.log(`[WebSocket] PC-${computerId} disconnected`);
        }
    });
    
    ws.on('error', (err) => {
        console.error('[WebSocket] Error:', err);
    });
    
    // Отправляем приветственное сообщение
    ws.send(JSON.stringify({
        type: 'welcome',
        server: 'ESports Arena Club Server',
        version: '1.0.0',
        clubId: config.clubId,
        timestamp: new Date().toISOString()
    }));
});

console.log(`[WebSocket] Server listening on port ${config.wsPort}`);

// ============================================
// 4. ЗАПУСК HTTP СЕРВЕРА
// ============================================

const start = async () => {
    try {
        await app.listen({ port: config.httpPort, host: '0.0.0.0' });
        console.log(`[HTTP] Server listening on http://0.0.0.0:${config.httpPort}`);
        console.log(`
╔══════════════════════════════════════════════════════════════╗
║     ESports Arena - Club Server (Test Version)              ║
╠══════════════════════════════════════════════════════════════╣
║  Club ID: ${config.clubId.toString().padEnd(44)}║
║  Club Name: ${config.clubName.padEnd(44)}║
║  HTTP API:  http://localhost:${config.httpPort}${' '.repeat(44 - (18 + config.httpPort.toString().length))}║
║  WebSocket: ws://localhost:${config.wsPort}${' '.repeat(44 - (19 + config.wsPort.toString().length))}║
║                                                              ║
║  📋 Available endpoints:                                    ║
║     GET  /          - Главная страница                      ║
║     GET  /health    - Проверка здоровья                     ║
║     GET  /status    - Детальный статус                      ║
║     GET  /pcs       - Список ПК                             ║
║     GET  /sessions  - Активные сессии                       ║
║     GET  /clients   - Список клиентов                       ║
║     POST /start-session - Начать сессию                     ║
║     POST /end-session   - Завершить сессию                  ║
║     POST /topup         - Пополнить баланс                  ║
║     POST /sync          - Синхронизация с Cloud             ║
║                                                              ║
║  🔌 WebSocket commands:                                     ║
║     register      - Регистрация ПК                          ║
║     start_session - Начать сессию                           ║
║     end_session   - Завершить сессию                        ║
║     heartbeat     - Проверка связи                          ║
╚══════════════════════════════════════════════════════════════╝
        `);
    } catch (err) {
        console.error('[HTTP] Error starting server:', err);
        process.exit(1);
    }
};

start();

// ============================================
// 5. ФОНОВЫЕ ЗАДАЧИ
// ============================================

// Эмуляция подключения тестовых ПК (для демонстрации)
setTimeout(() => {
    console.log('\n[EMULATION] Adding test PCs...');
    
    const testPCs = [1, 2, 3, 4, 5];
    testPCs.forEach(id => {
        if (!pcs.has(id)) {
            const testPC = {
                id: id,
                ip: `192.168.1.${100 + id}`,
                status: 'online',
                session: null,
                registeredAt: new Date()
            };
            pcs.set(id, testPC);
            console.log(`[EMULATION] PC-${id} added (${testPC.ip})`);
        }
    });
    
    console.log(`[EMULATION] Total PCs: ${pcs.size}`);
}, 2000);

// Эмуляция отправки статистики в Cloud (каждые 30 секунд)
setInterval(() => {
    console.log('\n[SYNC] Sending stats to cloud...');
    console.log(`  Active PCs: ${pcs.size}`);
    console.log(`  Active sessions: ${activeSessions.size}`);
    console.log(`  Total balance: ${stats.totalBalance} руб`);
    console.log(`  Total sessions today: ${stats.totalSessions}`);
}, 30000);

// Graceful shutdown
process.on('SIGINT', () => {
    console.log('\n[SHUTDOWN] Received SIGINT, closing server...');
    wss.close(() => {
        console.log('[SHUTDOWN] WebSocket server closed');
        app.close(() => {
            console.log('[SHUTDOWN] HTTP server closed');
            process.exit(0);
        });
    });
});

process.on('SIGTERM', () => {
    console.log('\n[SHUTDOWN] Received SIGTERM, closing server...');
    wss.close(() => {
        app.close(() => {
            process.exit(0);
        });
    });
});
```

---

## 🔧 .env.example

```env
# Club Server Configuration
CLUB_ID=1
CLUB_NAME=Test Gaming Club
HTTP_PORT=3000
WS_PORT=3001

# Cloud API (for sync)
CLOUD_API_URL=https://api.yourdomain.com
CLOUD_API_KEY=your-api-key-here

# Database (optional)
DB_PATH=/opt/club-server/cache.db
```

---

## 📝 README.md

```markdown
# ESports Arena - Club Server (Test Version)

## Установка

```bash
# 1. Клонировать проект
git clone ...
cd club-server-test

# 2. Установить зависимости
npm install

# 3. Настроить .env
cp .env.example .env
# Отредактировать .env при необходимости

# 4. Запустить
npm start

# Или в режиме разработки
npm run dev
```

## Тестирование

### HTTP API

```bash
# Главная страница
curl http://localhost:3000/

# Статус
curl http://localhost:3000/status

# Список ПК
curl http://localhost:3000/pcs

# Активные сессии
curl http://localhost:3000/sessions

# Начать сессию
curl -X POST http://localhost:3000/start-session \
  -H "Content-Type: application/json" \
  -d '{"computerId":1,"clientId":123}'

# Пополнить баланс
curl -X POST http://localhost:3000/topup \
  -H "Content-Type: application/json" \
  -d '{"clientId":123,"amount":500}'
```

### WebSocket (с помощью wscat)

```bash
# Установить wscat
npm install -g wscat

# Подключиться
wscat -c ws://localhost:3001

# Зарегистрировать ПК
> {"type":"register","computerId":1}

# Начать сессию
> {"type":"start_session","clientId":123,"balance":1000}

# Heartbeat
> {"type":"heartbeat"}
```

## Структура ответов

### GET /status
```json
{
  "club": {
    "id": 1,
    "name": "Test Gaming Club",
    "online": true
  },
  "stats": {
    "pcs": {
      "total": 5,
      "online": 5,
      "inGame": 2
    },
    "sessions": {
      "active": 2,
      "total": 15
    },
    "finances": {
      "totalBalance": 12500,
      "todayRevenue": 12500
    }
  },
  "pcs": [...],
  "sessions": [...]
}
```

## Команды WebSocket

| Команда | Описание |
|---------|----------|
| `register` | Регистрация ПК |
| `start_session` | Начать сессию |
| `end_session` | Завершить сессию |
| `heartbeat` | Проверка связи |

## Особенности тестовой версии

- Все данные хранятся в памяти
- Эмуляция ПК (автоматически создаются тестовые ПК 1-5)
- Логи в консоль
- Нет реального взаимодействия с Windows API
- Подходит для тестирования API и WebSocket

## Продакшен версия

Для продакшена добавить:
- SQLite для офлайн-кэша
- Реальное управление ПК через Windows API
- WebSocket клиент для связи с Cloud
- Упаковка в .exe через pkg
- Windows Service для автозапуска
```

---

## 🚀 Запуск и тестирование

```bash
# 1. Установка
npm install

# 2. Запуск
npm start

# 3. Открыть браузер
http://localhost:3000

# 4. Проверить API
curl http://localhost:3000/status

# 5. Подключиться по WebSocket (в другом терминале)
wscat -c ws://localhost:3001

# 6. Зарегистрировать ПК
{"type":"register","computerId":10}
```

---

## 📊 Что вы получите

1. **HTTP сервер на порту 3000** с красивым JSON API
2. **WebSocket сервер на порту 3001** для ПК
3. **Эмуляция 5 тестовых ПК** (автоматически)
4. **Заглушки для всех операций**
5. **Цветной консольный вывод** с ASCII логотипом
6. **Graceful shutdown** (Ctrl+C)

Этот скрипт можно использовать как:
- Тестовый стенд для разработки PC Client
- Демонстрацию архитектуры
- Основу для продакшен-версии (добавить SQLite, реальные команды ПК)

Теперь у тебя есть полностью рабочий тестовый Club Server! 🎮