# Инструкция по настройке и тестированию связки ESP8266 и Android-приложения
## 1. Настройка ESP8266
**Необходимое оборудование:**
- Модуль ESP8266 (например, NodeMCU).
- Счетчик Гейгера с цифровым выходом.
- Кабель micro-USB для подключения ESP8266 к компьютеру.
**Шаги:**
- Установите Arduino IDE
- Скачайте и установите Arduino IDE.
- Запустите IDE.
### 1.2. Добавьте поддержку ESP8266
- В Arduino IDE перейдите в File > Preferences.
- В поле Additional Board Manager URLs добавьте:
```bash
https://arduino.esp8266.com/stable/package_esp8266com_index.json
```
- Перейдите в Tools > Board > Boards Manager.
- Найдите и установите ESP8266.
### 1.3. Подключите ESP8266 к компьютеру
- Подключите модуль ESP8266 через USB.
- Выберите порт:
-- Перейдите в Tools > Port и выберите соответствующий COM-порт.
### 1.4. Загрузите код
Используйте следующий пример кода для сбора данных с подключенного счетчика Гейгера:

```cpp
#include <ESP8266WiFi.h>

const char* ssid = "ESP8266_AP";
const char* password = "12345678";

WiFiServer server(80);

volatile unsigned long pulseCount = 0;
unsigned long lastUpdateTime = 0;

float accumulatedRadiation = 0.0;
const float conversionFactor = 0.0057; // Коэффициент перевода в мкЗв/ч

void IRAM_ATTR handlePulse() {
    pulseCount++;
}

void setup() {
    Serial.begin(115200);
    WiFi.softAP(ssid, password);
    server.begin();

    pinMode(D1, INPUT_PULLUP);
    attachInterrupt(digitalPinToInterrupt(D1), handlePulse, RISING);

    Serial.println("ESP8266 initialized");
}

void loop() {
    if (millis() - lastUpdateTime >= 1000) {
        lastUpdateTime = millis();

        float currentRadiation = pulseCount * conversionFactor;
        accumulatedRadiation += currentRadiation / 3600.0;
        pulseCount = 0;
    }

    WiFiClient client = server.available();
    if (client) {
        String request = client.readStringUntil('\r');
        client.flush();

        if (request.startsWith("GET /data")) {
            String jsonResponse = "{\"currentRadiation\":" + String(pulseCount * conversionFactor) +
                                  ", \"accumulatedRadiation\":" + String(accumulatedRadiation) + "}";
            client.print("HTTP/1.1 200 OK\r\nContent-Type: application/json\r\n\r\n");
            client.print(jsonResponse);
        }
        client.stop();
    }
}
```
### 1.5. Загрузите скетч
- В Arduino IDE выберите Tools > Board > NodeMCU 1.0 (ESP-12E Module).
- Нажмите Upload.
## 2. Настройка Android-приложения
### 2.1. Подготовьте Android Studio
- Убедитесь, что Android Studio версии Giraffe (или выше).
- Импортируйте проект.
### 2.2. Настройка IP-адреса ESP
- В коде Android-приложения найдите строку, где указывается BASE_URL, и замените её на IP-адрес ESP8266 (например, 192.168.4.1).

```kotlin
private const val BASE_URL = "http://192.168.4.1"
```
### 2.3. Синхронизация Gradle
- Нажмите File > Sync Project with Gradle Files.
- Убедитесь, что синхронизация завершилась без ошибок.
### 2.4. Запустите приложение
- Подключите устройство через USB или запустите эмулятор.
- Нажмите Run > Run 'app'.
## 3. Тестирование связки ESP и Android
Для работы приложения телефон должен оставаться подключённым к сети Wi-Fi ESP8266, и отключение приведёт к прекращению получения данных.
### 3.1. Проверка ESP8266
- Подключитесь к Wi-Fi сети, созданной ESP8266:
- SSID: ESP8266_AP
- Пароль: 12345678.
- Откройте браузер и введите:
```arduino
http://192.168.4.1/data
```
Вы должны увидеть JSON-ответ, например:
```json
{
    "currentRadiation": 0.12,
    "accumulatedRadiation": 1.34
}
```
### 3.2. Проверка Android-приложения
- Убедитесь, что телефон подключён к той же Wi-Fi сети ESP8266.
- Запустите приложение.
Проверьте:
- Отображение текущей и накопленной радиации.
- Работа кнопок (сброс накопленной радиации, остановка таймера).
## 4. Решение возможных проблем
### 4.1. ESP8266 не подключается
- Проверьте подключение питания.
- Убедитесь, что выбран правильный COM-порт.
- Проверьте SSID и пароль.
### 4.2. Android-приложение не получает данные
- Убедитесь, что телефон подключён к Wi-Fi сети ESP8266.
- Проверьте IP-адрес ESP (выведите его в Serial Monitor с помощью WiFi.softAPIP()).
### 4.3. JSON не отображается в браузере
- Проверьте, правильно ли код ESP обрабатывает запросы.
- Проверьте, запущен ли сервер:
```cpp
server.begin();
```
### 4.4. Ошибки в Android Studio
- Убедитесь, что все зависимости установлены.
- Очистите и пересоберите проект:
Build > Clean Project.
Build > Rebuild Project.
