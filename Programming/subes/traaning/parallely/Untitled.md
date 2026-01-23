Отлично! Давайте разберем на конкретных практических заданиях с решениями. Я придумаю задачи разного уровня сложности и покажу, как их решать в PHP.

## **Задание 1: Параллельная обработка изображений**

**Задача:** Есть 1000 изображений, каждое нужно:
1. Изменить размер
2. Применить фильтр
3. Сохранить в новом формате

### **Решение 1: Процессы через pcntl (CPU-bound задача)**
```php
<?php

class ImageProcessor {
    private $workerCount;
    
    public function __construct(int $workerCount = 4) {
        $this->workerCount = $workerCount;
        if (!extension_loaded('pcntl')) {
            throw new RuntimeException('PCNTL extension required');
        }
    }
    
    public function processBatch(array $imagePaths): array {
        $chunks = array_chunk($imagePaths, ceil(count($imagePaths) / $this->workerCount));
        $pipes = [];
        $processes = [];
        $results = [];
        
        for ($i = 0; $i < $this->workerCount; $i++) {
            if (!isset($chunks[$i])) break;
            
            // Создаем каналы для общения с дочерним процессом
            $descriptorspec = [
                0 => ["pipe", "r"], // stdin
                1 => ["pipe", "w"], // stdout
                2 => ["pipe", "w"]  // stderr
            ];
            
            // Запускаем дочерний процесс
            $process = proc_open(
                'php ' . __DIR__ . '/image_worker.php',
                $descriptorspec,
                $pipes[$i]
            );
            
            if (!is_resource($process)) {
                throw new RuntimeException("Failed to start worker $i");
            }
            
            // Отправляем данные в дочерний процесс
            fwrite($pipes[$i][0], serialize($chunks[$i]));
            fclose($pipes[$i][0]);
            
            $processes[$i] = [
                'resource' => $process,
                'stdout' => $pipes[$i][1],
                'stderr' => $pipes[$i][2]
            ];
        }
        
        // Собираем результаты
        foreach ($processes as $i => $proc) {
            $output = stream_get_contents($proc['stdout']);
            $error = stream_get_contents($proc['stderr']);
            
            fclose($proc['stdout']);
            fclose($proc['stderr']);
            proc_close($proc['resource']);
            
            if (!empty($error)) {
                throw new RuntimeException("Worker $i error: $error");
            }
            
            $results = array_merge($results, unserialize($output));
        }
        
        return $results;
    }
}

// Файл image_worker.php
<?php
// Получаем данные из stdin
$input = stream_get_contents(STDIN);
$images = unserialize($input);

$results = [];
foreach ($images as $imagePath) {
    // Имитация тяжелой обработки
    usleep(rand(100000, 500000)); // 0.1-0.5 секунды
    
    $result = [
        'path' => $imagePath,
        'processed' => true,
        'new_path' => '/processed/' . basename($imagePath),
        'size' => filesize($imagePath)
    ];
    
    $results[] = $result;
}

// Возвращаем результат
echo serialize($results);
```

### **Решение 2: Parallel extension (требует установки)**
```php
<?php

class ParallelImageProcessor {
    public function processWithParallel(array $imagePaths): array {
        $runtime = new \parallel\Runtime();
        
        // Запускаем обработку в отдельном потоке
        $future = $runtime->run(function(array $images) {
            $results = [];
            foreach ($images as $image) {
                // Здесь реальная обработка
                $img = imagecreatefromjpeg($image);
                $img = imagescale($img, 800);
                $img = imagefilter($img, IMG_FILTER_GRAYSCALE);
                
                $newPath = '/processed/' . basename($image);
                imagejpeg($img, $newPath);
                
                $results[] = [
                    'original' => $image,
                    'processed' => $newPath,
                    'memory' => memory_get_peak_usage()
                ];
                
                imagedestroy($img);
            }
            return $results;
        }, [$imagePaths]);
        
        return $future->value(); // Блокирует до получения результата
    }
}
```

## **Задание 2: Параллельные HTTP-запросы**

**Задача:** Сделать 100 HTTP-запросов к API, собрать данные, обработать

### **Решение 1: ReactPHP (асинхронно, но в одном процессе)**
```php
<?php

require 'vendor/autoload.php';

class AsyncHttpFetcher {
    private $loop;
    private $httpClient;
    
    public function __construct() {
        $this->loop = \React\EventLoop\Factory::create();
        $this->httpClient = new \Clue\React\Http\Browser($this->loop);
    }
    
    public function fetchUrls(array $urls, int $concurrency = 10): array {
        $results = [];
        $promises = [];
        
        // Создаем ограничитель параллелизма
        $limiter = new \Clue\React\Mq\Queue($concurrency, null, 
            function($url) {
                return $this->httpClient->get($url);
            }
        );
        
        foreach ($urls as $url) {
            $promises[] = $limiter($url)->then(
                function (\Psr\Http\Message\ResponseInterface $response) use ($url, &$results) {
                    $data = json_decode((string)$response->getBody(), true);
                    $results[$url] = $this->processData($data);
                    return $results[$url];
                },
                function (Exception $e) use ($url, &$results) {
                    $results[$url] = ['error' => $e->getMessage()];
                    return null;
                }
            );
        }
        
        // Ждем завершения всех запросов
        \React\Promise\all($promises)->then(
            function () use (&$results) {
                $this->loop->stop();
            }
        );
        
        $this->loop->run();
        return $results;
    }
    
    private function processData(array $data): array {
        // Имитация обработки
        usleep(10000); // 10ms
        return [
            'processed' => true,
            'items' => count($data),
            'timestamp' => time()
        ];
    }
}

// Использование:
$fetcher = new AsyncHttpFetcher();
$urls = array_fill(0, 100, 'https://api.example.com/data');
$results = $fetcher->fetchUrls($urls, 20); // 20 одновременных запросов
```

### **Решение 2: Амп с корутинами**
```php
<?php

require 'vendor/autoload.php';

use Amp\Parallel\Worker;
use Amp\Future;

class AmpHttpFetcher {
    public function fetchParallel(array $urls, int $batchSize = 25): array {
        $batches = array_chunk($urls, $batchSize);
        $futures = [];
        
        foreach ($batches as $batchIndex => $batch) {
            $futures[] = Worker\submit(
                new class($batch) implements \Amp\Parallel\Worker\Task {
                    private $urls;
                    
                    public function __construct(array $urls) {
                        $this->urls = $urls;
                    }
                    
                    public function run(): array {
                        $results = [];
                        foreach ($this->urls as $url) {
                            $ch = curl_init($url);
                            curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
                            curl_setopt($ch, CURLOPT_TIMEOUT, 30);
                            $response = curl_exec($ch);
                            $results[$url] = [
                                'success' => $response !== false,
                                'data' => json_decode($response, true),
                                'error' => curl_error($ch)
                            ];
                            curl_close($ch);
                        }
                        return $results;
                    }
                }
            );
        }
        
        // Ждем все батчи
        $allResults = Future\await($futures);
        
        // Объединяем результаты
        $final = [];
        foreach ($allResults as $batchResults) {
            $final = array_merge($final, $batchResults);
        }
        
        return $final;
    }
}
```

## **Задание 3: Параллельная обработка CSV + запись в БД**

**Задача:** 
1. Прочитать CSV файл (1 млн строк)
2. Валидировать каждую строку
3. Сохранить в базу данных
4. Сгенерировать отчет

### **Решение: Процессный пул с очередями**
```php
<?php

class CsvProcessor {
    private $db;
    private $workerCount;
    
    public function __construct(\PDO $db, int $workerCount = 4) {
        $this->db = $db;
        $this->workerCount = $workerCount;
    }
    
    public function processLargeCsv(string $csvPath): array {
        // Шаг 1: Разделяем файл на части
        $chunkFiles = $this->splitCsv($csvPath, $this->workerCount);
        
        // Шаг 2: Запускаем воркеров
        $workers = [];
        $results = [];
        
        for ($i = 0; $i < $this->workerCount; $i++) {
            $pipePath = sys_get_temp_dir() . "/csv_pipe_$i";
            posix_mkfifo($pipePath, 0666);
            
            $pid = pcntl_fork();
            
            if ($pid == -1) {
                throw new RuntimeException("Could not fork");
            }
            
            if ($pid == 0) { // Дочерний процесс
                $this->workerProcess($chunkFiles[$i], $pipePath, $i);
                exit(0);
            } else { // Родительский процесс
                $workers[$pid] = $pipePath;
            }
        }
        
        // Шаг 3: Собираем результаты через именованные каналы
        foreach ($workers as $pid => $pipePath) {
            $pipe = fopen($pipePath, 'r');
            $workerResult = unserialize(stream_get_contents($pipe));
            fclose($pipe);
            
            $results[] = $workerResult;
            
            // Очистка
            unlink($pipePath);
            
            // Ждем завершения процесса
            pcntl_waitpid($pid, $status);
        }
        
        // Шаг 4: Агрегируем финальный отчет
        return $this->generateReport($results);
    }
    
    private function workerProcess(string $chunkFile, string $pipePath, int $workerId): void {
        $pipe = fopen($pipePath, 'w');
        
        $results = [
            'processed' => 0,
            'errors' => 0,
            'valid' => 0,
            'worker_id' => $workerId
        ];
        
        $handle = fopen($chunkFile, 'r');
        if ($handle) {
            while (($row = fgetcsv($handle)) !== false) {
                $results['processed']++;
                
                if ($this->validateRow($row)) {
                    $this->saveToDatabase($row);
                    $results['valid']++;
                } else {
                    $results['errors']++;
                }
            }
            fclose($handle);
        }
        
        // Отправляем результат в родительский процесс
        fwrite($pipe, serialize($results));
        fclose($pipe);
        
        // Удаляем временный файл
        unlink($chunkFile);
    }
    
    private function splitCsv(string $csvPath, int $parts): array {
        $lineCount = 0;
        $handle = fopen($csvPath, 'r');
        while (!feof($handle)) {
            fgets($handle);
            $lineCount++;
        }
        fclose($handle);
        
        $linesPerPart = ceil($lineCount / $parts);
        $chunkFiles = [];
        $currentPart = 0;
        $currentLine = 0;
        
        $handle = fopen($csvPath, 'r');
        $currentFile = null;
        
        while (($line = fgets($handle)) !== false) {
            if ($currentLine % $linesPerPart == 0) {
                if ($currentFile) fclose($currentFile);
                
                $chunkFile = tempnam(sys_get_temp_dir(), 'csv_chunk_');
                $currentFile = fopen($chunkFile, 'w');
                $chunkFiles[] = $chunkFile;
                $currentPart++;
            }
            
            fwrite($currentFile, $line);
            $currentLine++;
        }
        
        if ($currentFile) fclose($currentFile);
        fclose($handle);
        
        return $chunkFiles;
    }
    
    private function validateRow(array $row): bool {
        // Логика валидации
        return count($row) >= 3 && filter_var($row[1], FILTER_VALIDATE_EMAIL);
    }
    
    private function saveToDatabase(array $row): void {
        $stmt = $this->db->prepare(
            "INSERT INTO users (name, email, age) VALUES (?, ?, ?)"
        );
        $stmt->execute([$row[0], $row[1], $row[2]]);
    }
    
    private function generateReport(array $workerResults): array {
        $total = [
            'total_processed' => 0,
            'total_valid' => 0,
            'total_errors' => 0,
            'workers' => $workerResults
        ];
        
        foreach ($workerResults as $result) {
            $total['total_processed'] += $result['processed'];
            $total['total_valid'] += $result['valid'];
            $total['total_errors'] += $result['errors'];
        }
        
        $total['success_rate'] = $total['total_processed'] > 0 
            ? ($total['total_valid'] / $total['total_processed']) * 100 
            : 0;
            
        return $total;
    }
}
```

## **Задание 4: Реактивный WebSocket сервер с брокером сообщений**

### **Решение: Swoole + Redis Pub/Sub**
```php
<?php

class WebSocketServer {
    private $server;
    private $redis;
    private $rooms = [];
    
    public function __construct(string $host = '0.0.0.0', int $port = 9501) {
        $this->server = new Swoole\WebSocket\Server($host, $port);
        $this->redis = new Redis();
        $this->redis->connect('127.0.0.1', 6379);
        
        $this->setupEvents();
    }
    
    private function setupEvents(): void {
        $this->server->on('start', function($server) {
            echo "WebSocket Server started at ws://{$server->host}:{$server->port}\n";
            
            // Запускаем обработчик Redis сообщений в отдельном процессе
            $this->startRedisSubscriber();
        });
        
        $this->server->on('open', function($server, $request) {
            echo "Connection open: {$request->fd}\n";
            
            // Параллельная аутентификация
            go(function() use ($server, $request) {
                $user = $this->authenticateAsync($request);
                if ($user) {
                    $server->push($request->fd, json_encode([
                        'type' => 'welcome',
                        'user' => $user
                    ]));
                }
            });
        });
        
        $this->server->on('message', function($server, $frame) {
            // Обработка сообщения в отдельной корутине
            go(function() use ($server, $frame) {
                $data = json_decode($frame->data, true);
                
                // Параллельная обработка разных типов сообщений
                switch ($data['type'] ?? '') {
                    case 'join_room':
                        $this->joinRoom($frame->fd, $data['room']);
                        break;
                    case 'broadcast':
                        $this->broadcastMessage($frame->fd, $data);
                        break;
                    case 'private':
                        $this->sendPrivateMessage($frame->fd, $data);
                        break;
                }
            });
        });
        
        $this->server->on('close', function($server, $fd) {
            echo "Connection closed: {$fd}\n";
            $this->leaveAllRooms($fd);
        });
    }
    
    private function startRedisSubscriber(): void {
        // Запускаем в отдельном процессе подписку на Redis
        $process = new Swoole\Process(function($process) {
            $redis = new Redis();
            $redis->connect('127.0.0.1', 6379);
            $redis->subscribe(['websocket_broadcast'], function($redis, $channel, $message) {
                $data = json_decode($message, true);
                
                // Рассылка всем подключенным клиентам
                foreach ($this->server->connections as $fd) {
                    if ($this->server->isEstablished($fd)) {
                        $this->server->push($fd, $message);
                    }
                }
            });
        });
        
        $this->server->addProcess($process);
    }
    
    private function authenticateAsync($request): ?array {
        // Имитация асинхронной аутентификации
        co::sleep(0.1); // Неблокирующая задержка
        
        $token = $request->get['token'] ?? '';
        if ($token) {
            // Проверка в базе данных (асинхронно)
            $dbResult = $this->queryDatabaseAsync("SELECT * FROM users WHERE token = ?", [$token]);
            return $dbResult[0] ?? null;
        }
        
        return null;
    }
    
    private function queryDatabaseAsync(string $query, array $params): array {
        // Асинхронный запрос к БД через пул соединений
        $mysql = new Swoole\Coroutine\MySQL();
        $mysql->connect([
            'host' => '127.0.0.1',
            'port' => 3306,
            'user' => 'user',
            'password' => 'pass',
            'database' => 'db'
        ]);
        
        $stmt = $mysql->prepare($query);
        return $stmt->execute($params);
    }
    
    private function joinRoom(int $fd, string $room): void {
        if (!isset($this->rooms[$room])) {
            $this->rooms[$room] = [];
        }
        
        $this->rooms[$room][$fd] = true;
        
        // Уведомляем всех в комнате (параллельно)
        $message = json_encode([
            'type' => 'user_joined',
            'room' => $room,
            'user_id' => $fd
        ]);
        
        foreach ($this->rooms[$room] as $roomFd => $_) {
            if ($roomFd != $fd) {
                go(function() use ($roomFd, $message) {
                    $this->server->push($roomFd, $message);
                });
            }
        }
    }
    
    private function broadcastMessage(int $fd, array $data): void {
        $room = $data['room'] ?? 'general';
        
        if (isset($this->rooms[$room][$fd])) {
            $broadcastData = [
                'type' => 'message',
                'room' => $room,
                'from' => $fd,
                'text' => $data['text'],
                'timestamp' => time()
            ];
            
            // Публикуем в Redis для горизонтального масштабирования
            $this->redis->publish('websocket_broadcast', json_encode($broadcastData));
        }
    }
    
    public function start(): void {
        $this->server->start();
    }
}

// Запуск сервера
$server = new WebSocketServer();
$server->start();
```

## **Задание 5: Распределенный Map-Reduce**

**Задача:** Посчитать частоту слов в 1000 текстовых файлах

### **Решение: RabbitMQ + Workers**
```php
<?php

class MapReduceProcessor {
    private $connection;
    private $channel;
    
    public function __construct() {
        $this->connection = new \PhpAmqpLib\Connection\AMQPStreamConnection(
            'localhost', 5672, 'guest', 'guest'
        );
        $this->channel = $this->connection->channel();
        
        // Очереди для Map и Reduce
        $this->channel->queue_declare('map_tasks', false, true, false, false);
        $this->channel->queue_declare('reduce_tasks', false, true, false, false);
        $this->channel->queue_declare('results', false, true, false, false);
    }
    
    public function processFiles(array $filePaths): array {
        // Шаг 1: Отправляем задачи на Map
        $this->sendMapTasks($filePaths);
        
        // Шаг 2: Запускаем Map workers
        $this->startMapWorkers(4);
        
        // Шаг 3: Запускаем Reduce worker
        $this->startReduceWorker();
        
        // Шаг 4: Ждем результат
        return $this->getFinalResult();
    }
    
    private function sendMapTasks(array $filePaths): void {
        foreach ($filePaths as $filePath) {
            $message = new \PhpAmqpLib\Message\AMQPMessage(
                json_encode(['file' => $filePath]),
                ['delivery_mode' => 2] // Сохранять при перезапуске
            );
            $this->channel->basic_publish($message, '', 'map_tasks');
        }
    }
    
    private function startMapWorkers(int $count): void {
        for ($i = 0; $i < $count; $i++) {
            $pid = pcntl_fork();
            
            if ($pid == 0) { // Дочерний процесс (worker)
                $this->mapWorker();
                exit(0);
            }
        }
    }
    
    private function mapWorker(): void {
        $connection = new \PhpAmqpLib\Connection\AMQPStreamConnection(
            'localhost', 5672, 'guest', 'guest'
        );
        $channel = $connection->channel();
        
        $callback = function($msg) use ($channel) {
            $data = json_decode($msg->body, true);
            $result = $this->map($data['file']);
            
            // Отправляем результат в очередь для Reduce
            $resultMsg = new \PhpAmqpLib\Message\AMQPMessage(
                json_encode($result),
                ['delivery_mode' => 2]
            );
            $channel->basic_publish($resultMsg, '', 'reduce_tasks');
            
            $channel->basic_ack($msg->delivery_info['delivery_tag']);
        };
        
        $channel->basic_qos(null, 1, null); // По одной задаче за раз
        $channel->basic_consume('map_tasks', '', false, false, false, false, $callback);
        
        while (count($channel->callbacks)) {
            $channel->wait();
        }
    }
    
    private function map(string $filePath): array {
        $text = file_get_contents($filePath);
        $words = str_word_count(strtolower($text), 1);
        
        $wordCount = [];
        foreach ($words as $word) {
            $wordCount[$word] = ($wordCount[$word] ?? 0) + 1;
        }
        
        return [
            'file' => $filePath,
            'word_count' => $wordCount,
            'total_words' => count($words)
        ];
    }
    
    private function startReduceWorker(): void {
        $pid = pcntl_fork();
        
        if ($pid == 0) {
            $this->reduceWorker();
            exit(0);
        }
    }
    
    private function reduceWorker(): void {
        $connection = new \PhpAmqpLib\Connection\AMQPStreamConnection(
            'localhost', 5672, 'guest', 'guest'
        );
        $channel = $connection->channel();
        
        $globalResult = [];
        $processedFiles = 0;
        
        $callback = function($msg) use (&$globalResult, &$processedFiles, $channel) {
            $data = json_decode($msg->body, true);
            
            // Reduce: объединяем результаты
            foreach ($data['word_count'] as $word => $count) {
                $globalResult[$word] = ($globalResult[$word] ?? 0) + $count;
            }
            
            $processedFiles++;
            
            // Если обработали все файлы, отправляем финальный результат
            if ($processedFiles >= 1000) {
                arsort($globalResult);
                $final = array_slice($globalResult, 0, 100); // Топ-100 слов
                
                $resultMsg = new \PhpAmqpLib\Message\AMQPMessage(
                    json_encode($final),
                    ['delivery_mode' => 2]
                );
                $channel->basic_publish($resultMsg, '', 'results');
            }
            
            $channel->basic_ack($msg->delivery_info['delivery_tag']);
        };
        
        $channel->basic_consume('reduce_tasks', '', false, false, false, false, $callback);
        
        while (count($channel->callbacks)) {
            $channel->wait();
        }
    }
    
    private function getFinalResult(): array {
        $this->channel->basic_consume('results', '', false, true, false, false, 
            function($msg) use (&$result) {
                $result = json_decode($msg->body, true);
            }
        );
        
        while (!$result) {
            $this->channel->wait();
        }
        
        return $result;
    }
}
```

## **Сравнение подходов для разных задач:**

| Задача | Лучшее решение | Почему | Производительность |
|--------|---------------|---------|-------------------|
| Обработка изображений | Parallel + процессы | CPU-bound, нужна изоляция | Высокая |
| HTTP-запросы | ReactPHP/Amp | IO-bound, много соединений | Очень высокая |
| Обработка CSV | pcntl + каналы | Работа с файлами, контроль памяти | Средняя |
| WebSocket | Swoole | Событийная модель, корутины | Максимальная |
| Map-Reduce | RabbitMQ + workers | Распределенные вычисления | Масштабируемая |

## **Ключевые выводы:**

1. **CPU-bound задачи** → процессы или parallel extension
2. **IO-bound задачи** → асинхронность (ReactPHP, Swoole, Amp)
3. **Распределенные задачи** → очередь сообщений (RabbitMQ, Redis)
4. **Важно учитывать**:
   - Стоимость создания процессов
   - Объем памяти
   - Сложность синхронизации
   - Управление ошибками

Хотите, чтобы я подробнее разобрал какой-то конкретный кейс или добавил больше примеров с профилированием производительности?