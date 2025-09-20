Отлично! Вот как будет выглядеть полная архитектура:

## 🏗️ Архитектура системы

```
[Flutter App] → [FastAPI Server] → [PaddleOCR] → [JSON Parser] → [Flutter App]
       ↑                ↑               ↑             ↑               ↑
   (фото чека)     (прием image)   (OCR обработка) (структурирование) (результат)
```

---

## 📁 Структура проекта

```
receipt-ai-server/
├── app/
│   ├── main.py              # FastAPI приложение
│   ├── models.py           # Модели данных
│   ├── ocr_processor.py    # OCR обработчик
│   ├── receipt_parser.py   # Парсер чека
│   └── utils.py           # Вспомогательные функции
├── requirements.txt
└── Dockerfile
```

---

## 🧩 1. Модели данных (models.py)

```python
from pydantic import BaseModel
from typing import List, Optional
from datetime import datetime

class ReceiptItem(BaseModel):
    name: str
    quantity: float
    price: float
    total: float
    discount: Optional[float] = 0.0

class ReceiptData(BaseModel):
    store_name: Optional[str] = None
    store_address: Optional[str] = None
    date: Optional[str] = None
    total_amount: float
    items: List[ReceiptItem]
    tax_amount: Optional[float] = 0.0
    discount_amount: Optional[float] = 0.0
    payment_method: Optional[str] = None
    confidence: float  # Общая уверенность распознавания

class ProcessReceiptResponse(BaseModel):
    success: bool
    data: Optional[ReceiptData] = None
    error: Optional[str] = None
    processing_time: float
```

---

## 🔧 2. OCR процессор (ocr_processor.py)

```python
from paddleocr import PaddleOCR
import cv2
import numpy as np
import logging
from typing import List, Tuple

class OCRProcessor:
    def __init__(self):
        self.ocr = PaddleOCR(
            use_angle_cls=True,
            lang='ru',
            use_gpu=False,  # True для GPU
            show_log=False
        )
        self.logger = logging.getLogger(__name__)
    
    async def process_image(self, image_data: bytes) -> List[Tuple]:
        try:
            # Конвертируем bytes в numpy array
            nparr = np.frombuffer(image_data, np.uint8)
            img = cv2.imdecode(nparr, cv2.IMREAD_COLOR)
            
            # Предобработка изображения
            img = self._preprocess_image(img)
            
            # OCR обработка
            result = self.ocr.ocr(img, cls=True)
            
            return result[0] if result else []
            
        except Exception as e:
            self.logger.error(f"OCR processing error: {e}")
            raise

    def _preprocess_image(self, img):
        """Улучшаем качество изображения для OCR"""
        # Конвертируем в grayscale
        gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
        
        # Увеличиваем контраст
        clahe = cv2.createCLAHE(clipLimit=2.0, tileGridSize=(8,8))
        enhanced = clahe.apply(gray)
        
        # Убираем шум
        denoised = cv2.medianBlur(enhanced, 3)
        
        return denoised
```

---

## 🧠 3. Парсер чека (receipt_parser.py)

```python
import re
from typing import List, Dict, Any
from .models import ReceiptData, ReceiptItem
from datetime import datetime

class ReceiptParser:
    def __init__(self):
        self.price_pattern = r'(\d+[\.,]\d{2})'
        self.date_patterns = [
            r'\d{2}\.\d{2}\.\d{4}',
            r'\d{2}/\d{2}/\d{4}',
            r'\d{4}-\d{2}-\d{2}'
        ]
    
    def parse_ocr_result(self, ocr_result: List, processing_time: float) -> ReceiptData:
        lines = self._extract_text_lines(ocr_result)
        
        store_name = self._find_store_name(lines)
        date = self._find_date(lines)
        items = self._extract_items(lines)
        total_amount = self._find_total_amount(lines)
        
        return ReceiptData(
            store_name=store_name,
            date=date,
            total_amount=total_amount,
            items=items,
            confidence=self._calculate_confidence(ocr_result),
            processing_time=processing_time
        )
    
    def _extract_text_lines(self, ocr_result: List) -> List[Dict]:
        lines = []
        for line in ocr_result:
            if line:
                text = line[1][0]
                confidence = line[1][1]
                bbox = line[0]
                lines.append({
                    'text': text,
                    'confidence': confidence,
                    'bbox': bbox,
                    'y_position': sum([point[1] for point in bbox]) / 4  # Средняя Y координата
                })
        return sorted(lines, key=lambda x: x['y_position'])
    
    def _find_store_name(self, lines: List[Dict]) -> str:
        for line in lines:
            text = line['text'].lower()
            if any(keyword in text for keyword in ['магазин', 'супермаркет', 'торговый', 'сеть']):
                return line['text']
        return "Неизвестный магазин"
    
    def _extract_items(self, lines: List[Dict]) -> List[ReceiptItem]:
        items = []
        for line in lines:
            text = line['text']
            if self._is_product_line(text):
                item = self._parse_product_line(text, line['confidence'])
                if item:
                    items.append(item)
        return items
    
    def _is_product_line(self, text: str) -> bool:
        # Исключаем итоговые строки и служебную информацию
        exclude_keywords = ['итого', 'total', 'сумма', 'налог', 'скидка', 'чек']
        if any(keyword in text.lower() for keyword in exclude_keywords):
            return False
        
        # Ищем паттерны цен
        prices = re.findall(self.price_pattern, text)
        return len(prices) >= 1 and len(text) > 5
    
    def _parse_product_line(self, text: str, confidence: float) -> Optional[ReceiptItem]:
        try:
            # Упрощенный парсинг - нужно дорабатывать под конкретные чеки
            parts = text.split()
            prices = re.findall(self.price_pattern, text)
            
            if not prices:
                return None
                
            price = float(prices[-1].replace(',', '.'))
            name = ' '.join(parts[:-1])  # Все кроме последней части (цены)
            
            return ReceiptItem(
                name=name.strip(),
                quantity=1.0,
                price=price,
                total=price,
                discount=0.0
            )
            
        except (ValueError, IndexError):
            return None
    
    def _find_total_amount(self, lines: List[Dict]) -> float:
        for line in reversed(lines):  # Ищем с конца чека
            text = line['text'].lower()
            if 'итого' in text or 'total' in text or 'сумма' in text:
                prices = re.findall(self.price_pattern, line['text'])
                if prices:
                    return float(prices[-1].replace(',', '.'))
        return 0.0
    
    def _calculate_confidence(self, ocr_result: List) -> float:
        if not ocr_result:
            return 0.0
        confidences = [line[1][1] for line in ocr_result if line]
        return sum(confidences) / len(confidences) if confidences else 0.0
```

---

## 🚀 4. Главное приложение (main.py)

```python
from fastapi import FastAPI, File, UploadFile, HTTPException
from fastapi.middleware.cors import CORSMiddleware
from fastapi.responses import JSONResponse
import time
import logging
from typing import List

from .models import ProcessReceiptResponse
from .ocr_processor import OCRProcessor
from .receipt_parser import ReceiptParser

app = FastAPI(title="Receipt OCR API", version="1.0.0")

# CORS для Flutter приложения
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # В продакшене заменить на конкретные домены
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Инициализация компонентов
ocr_processor = OCRProcessor()
receipt_parser = ReceiptParser()

@app.post("/process-receipt", response_model=ProcessReceiptResponse)
async def process_receipt(image: UploadFile = File(...)):
    start_time = time.time()
    
    try:
        # Проверяем тип файла
        if not image.content_type.startswith('image/'):
            raise HTTPException(status_code=400, detail="Invalid image format")
        
        # Читаем изображение
        image_data = await image.read()
        
        # OCR обработка
        ocr_result = await ocr_processor.process_image(image_data)
        
        if not ocr_result:
            return ProcessReceiptResponse(
                success=False,
                error="Не удалось распознать текст на изображении",
                processing_time=time.time() - start_time
            )
        
        # Парсим результат
        processing_time = time.time() - start_time
        receipt_data = receipt_parser.parse_ocr_result(ocr_result, processing_time)
        
        return ProcessReceiptResponse(
            success=True,
            data=receipt_data,
            processing_time=processing_time
        )
        
    except Exception as e:
        logging.error(f"Processing error: {e}")
        return ProcessReceiptResponse(
            success=False,
            error=str(e),
            processing_time=time.time() - start_time
        )

@app.get("/health")
async def health_check():
    return {"status": "healthy", "service": "receipt-ocr-api"}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

---

## 📋 Пример ответа JSON

```json
{
  "success": true,
  "data": {
    "store_name": "Магнит",
    "store_address": null,
    "date": "15.12.2023",
    "total_amount": 1450.50,
    "items": [
      {
        "name": "Молоко Простоквашино",
        "quantity": 1,
        "price": 85.00,
        "total": 85.00,
        "discount": 0.0
      },
      {
        "name": "Хлеб Бородинский",
        "quantity": 1,
        "price": 45.50,
        "total": 45.50,
        "discount": 0.0
      }
    ],
    "tax_amount": 145.05,
    "discount_amount": 0.0,
    "payment_method": "CARD",
    "confidence": 0.89
  },
  "error": null,
  "processing_time": 2.45
}
```

---

## 🚀 Запуск сервера

```bash
# Установка зависимостей
pip install -r requirements.txt

# Запуск сервера
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

Теперь ваш Flutter клиент может отправлять POST запросы на `/process-receipt` с изображением чека и получать структурированный JSON ответ! 🎉