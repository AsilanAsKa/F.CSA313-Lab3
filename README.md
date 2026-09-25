# F.CSA313 Lab 3

**Оюутан:** Асилан Серикбай
**Оюутны код:** B232270115
**k6 version:** v2.2.0

## 1. Лабораторийн зорилго

Энэ лабораториор Node.js + Express API дээр k6 ашиглан Performance, Reliability болон Availability тест хийсэн. Мөн Quality Scenario, SLO, threshold, error budget болон chaos test ашиглаж системийн чанарыг шалгасан. Эцэст нь threshold-ийг зориудаар буруу тохируулж failure test хийсэн.

## 2. API

Энэ лабораторид Node.js + Express ашигласан.

Үндсэн API:

* `POST /cart/add` — сагсанд бараа нэмнэ.
* `GET /report` — report мэдээлэл буцаана.
* `POST /pay` — payment хийж, зарим хүсэлтэд 500 error буцаана.

Server:

```text
http://localhost:3000
```

## 3. Quality Scenarios

### 3.1 Performance — `/cart/add`

| 6 хэсэг                | Тайлбар                                                                      |
| ---------------------- | ---------------------------------------------------------------------------- |
| **Тойм**               | Хэрэглэгч сагсанд бараа нэмэх үед систем хурдан хариу өгөх чадварыг шалгана. |
| **Системийн төлөв**    | Express API ажиллаж, `/cart/add` endpoint хүсэлт хүлээн авч байна.           |
| **Орчны төлөв**        | Local орчинд 20 VU хэрэглэгч 2 минутын турш ажиллана.                        |
| **Гадаад өдөөлт**      | Хэрэглэгч `POST /cart/add` хүсэлт илгээнэ.                                   |
| **Шаардлагатай хариу** | Систем HTTP 200 status болон амжилттай response буцаана.                     |
| **Хэмжүүр**            | Response time **p95 < 200 ms**, **p99 < 300 ms** байна.                      |

### 3.2 Reliability — `/pay`

| 6 хэсэг                | Тайлбар                                                                               |
| ---------------------- | ------------------------------------------------------------------------------------- |
| **Тойм**               | Хэрэглэгч payment хийх үед систем алдаа багатай, найдвартай ажиллах чадварыг шалгана. |
| **Системийн төлөв**    | Express API ажиллаж, `/pay` endpoint хүсэлт хүлээн авч байна.                         |
| **Орчны төлөв**        | Local орчинд 20 VU хэрэглэгч 2 минутын турш payment request илгээнэ.                  |
| **Гадаад өдөөлт**      | Хэрэглэгч `POST /pay` хүсэлт илгээнэ.                                                 |
| **Шаардлагатай хариу** | Амжилттай payment үед HTTP 200 status болон `paid: true` response буцаана.            |
| **Хэмжүүр**            | Payment **error rate < 8%**, нийт **checks > 90%** байна.                             |

### 3.3 Availability — Server crash

| 6 хэсэг                | Тайлбар                                                                                                                     |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| **Тойм**               | Server түр зогссон үед систем хүсэлтүүдийг хэр хэмжээнд амжилттай боловсруулахыг шалгана.                                   |
| **Системийн төлөв**    | Test эхлэхэд Express API хэвийн ажиллаж байна.                                                                              |
| **Орчны төлөв**        | Local орчинд 20 VU хэрэглэгч 2 минутын турш API request илгээнэ.                                                            |
| **Гадаад өдөөлт**      | Test ажиллаж байх үед Express server-ийг зориудаар **10 секунд** зогсооно.                                                  |
| **Шаардлагатай хариу** | Server дахин ассаны дараа API request-үүд хэвийн ажиллаж, амжилттай response буцаана.                                       |
| **Хэмжүүр**            | Availability **> 90%**, `/cart/add` p95 **< 200 ms**, p99 **< 300 ms**, `/report` p95 **< 450 ms**, p99 **< 500 ms** байна. |

## 4. SLO

SLO нь системийн хүрэх ёстой үйлчилгээний түвшинг тодорхойлно.

| Сценарио     | SLI / Хэмжүүр             | Босго                      | Цонх / нөхцөл                            | Босго сонгосон үндэслэл                                                                                |
| ------------ | ------------------------- | -------------------------- | ---------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| Performance  | `/cart/add` response time | p95 < 200 ms, p99 < 300 ms | 20 VU, 2 минут                           | Cart-д бараа нэмэх үйлдэл хэрэглэгчид хурдан мэдрэгдэх шаардлагатай тул бага latency босго сонгосон.   |
| Reliability  | `/pay` error rate         | < 8%                       | 20 VU, 2 минут                           | Payment-ийн алдааг бага түвшинд байлгах шаардлагатай тул error rate-ийн 8%-ийн босго сонгосон.         |
| Availability | Successful checks         | > 90%                      | 20 VU, 2 минут, server 10 секунд зогсоно | Server түр зогссон үед ч хүсэлтүүдийн ихэнх нь амжилттай байх шаардлагатай тул 90%-ийн босго сонгосон. |
| Performance  | `/report` response time   | p95 < 450 ms, p99 < 500 ms | 20 VU, 2 минут                           | Report endpoint-ийн response time-ийг хянахын тулд p95 болон p99 latency босго сонгосон.               |

### Error Budget

Availability SLO:

```text
90%
```

Test-ийн хугацаа:

```text
2 минут = 120 секунд
```

Time-based error budget:

```text
120 × (1 - 0.90) = 12 секунд
```

Тиймээс 2 минутын test-ийн үед системийн зөвшөөрөгдөх downtime нь **12 секунд** байна.

### 12 секундийн budget болон request-based availability-ийн ялгаа

Time-based error budget нь server нийт хэдэн секунд ажиллаагүйг хэмждэг.

Chaos test-д server:

```text
10 секунд
```

зогсоосон.

Тиймээс:

```text
10 секунд < 12 секунд
```

болсон бөгөөд time-based error budget дотор байна.

Харин request-based availability нь нийт request-ээс хэд нь амжилттай болсныг хэмждэг. Server 10 секунд зогссон үед 20 VU зэрэг request илгээж байсан тул богино хугацаанд олон request failed болсон. Мөн `/pay` endpoint-ийн зарим request server ажиллаж байх үед ч 500 error өгсөн. Иймээс request-based availability нь time-based availability-аас бага гарсан.

Багшийн өмнө шалгасан chaos test-ийн request-based availability:

```text
84.28%
```

байсан.

Өөрөөр хэлбэл **10 секундын downtime нь 12 секундын time-based error budget дотор байсан боловч олон request failure болон payment error-оос шалтгаалан request-based availability 90%-ийн SLO-д хүрээгүй**.

Сүүлийн дахин ажиллуулсан chaos test-д request-based availability:

```text
85.72%
```

гарсан. Энэ нь request бүрийн random үр дүнгээс шалтгаалан test run бүрт бага зэрэг өөрчлөгдөж болохыг харуулж байна.

## 5. Pass Test

Baseline test:

```bash
k6 run --summary-trend-stats="avg,min,med,max,p(90),p(95),p(99)" slo-test.js 2>&1 | tee results/pass.txt
```

Сүүлийн pass test-ийн үр дүн:

| Metric         |    Result |
| -------------- | --------: |
| Checks         |    98.18% |
| Cart p95       |   1.62 ms |
| Cart p99       |  17.47 ms |
| Report p95     | 393.16 ms |
| Report p99     | 399.40 ms |
| Pay error rate |     5.45% |
| HTTP failed    |     1.81% |
| Requests       |      5550 |
| Iterations     |      1850 |

Threshold-тэй харьцуулахад:

```text
Cart p95 = 1.62 ms < 200 ms
Cart p99 = 17.47 ms < 300 ms

Report p95 = 393.16 ms < 450 ms
Report p99 = 399.40 ms < 500 ms

Pay error rate = 5.45% < 8%

Checks = 98.18% > 90%
```

Тиймээс pass test-ийн бүх threshold шаардлага хангасан.

Бүрэн output:

```text
results/pass.txt
```

## 6. Chaos Test

Chaos test хийхдээ k6 test ажиллаж байх үед Express server-ийг зориудаар **10 секунд** зогсоосон.

Сүүлийн chaos test-ийн үр дүн:

| Metric         |    Result |
| -------------- | --------: |
| Checks         |    85.72% |
| Cart p95       |   1.95 ms |
| Cart p99       |  23.48 ms |
| Report p95     | 389.46 ms |
| Report p99     | 398.35 ms |
| Pay error rate |    17.60% |
| HTTP failed    |    14.27% |
| Requests       |      5709 |
| Iterations     |      1903 |

Threshold-тэй харьцуулахад:

```text
Checks = 85.72% < 90%
```

тул availability threshold fail болсон.

Мөн:

```text
Pay error rate = 17.60% > 8%
```

тул reliability threshold fail болсон.

Харин performance threshold-үүд pass болсон:

```text
Cart p95 = 1.95 ms < 200 ms
Cart p99 = 23.48 ms < 300 ms

Report p95 = 389.46 ms < 450 ms
Report p99 = 398.35 ms < 500 ms
```

Иймээс chaos test-ийн үед availability болон reliability threshold fail болсон боловч performance threshold-үүд pass болсон.

Бүрэн output:

```text
results/chaos.txt
```

## 7. Intentional Threshold Failure Test

Threshold failure шалгахын тулд `slo-test-fail.js` тусдаа файл ашигласан.

Энэ файлд `/report` endpoint-ийн p95 threshold-ийг зориуд:

```text
p(95) < 100 ms
```

гэж өөрчилсөн.

Сүүлийн failure test-ийн үр дүн:

```text
Cart p95 = 1.62 ms
Cart p99 = 21.81 ms

Report p95 = 391.08 ms
Report p99 = 399.44 ms

Pay error rate = 3.61%
Checks = 98.79%
```

`/report`-ийн бодит p95:

```text
391.08 ms
```

харин шаардсан threshold:

```text
100 ms
```

байсан.

Тиймээс:

```text
391.08 ms > 100 ms
```

болж threshold fail болсон.

k6 threshold failure-ийн дараа:

```text
exit=99
```

гарсан.

Энэ нь threshold-ийг зориудаар зөрчүүлэхэд k6 test failure болж байгааг баталсан.

Бүрэн output:

```text
results/fail.txt
```

## 8. Файлууд

```text
lab03/
├── .gitignore
├── package.json
├── package-lock.json
├── server.js
├── slo-test.js
├── slo-test-fail.js
├── README.md
└── results/
    ├── pass.txt
    ├── chaos.txt
    └── fail.txt
```

`node_modules/`-ийг `.gitignore` файлд оруулсан.

## 9. Дүгнэлт

Энэ лабораториор k6 ашиглан API-ийн performance, reliability болон availability-г шалгасан. Quality Scenario бүрийг Тойм, Системийн төлөв, Орчны төлөв, Гадаад өдөөлт, Шаардлагатай хариу, Хэмжүүр гэсэн 6 хэсгээр тодорхойлсон. Pass test-ийн үед cart болон report endpoint-ийн latency, pay error rate болон checks-ийн threshold-үүд шаардлага хангасан. Cart-ийн p95 1.62 ms, report-ийн p95 393.16 ms байсан. Chaos test хийх үед server-ийг 10 секунд зогсоосноор checks 85.72% болж availability threshold fail болсон. Мөн pay error rate 17.60% болж reliability threshold fail болсон. Харин cart болон report-ийн latency threshold-үүд chaos test-ийн үед мөн pass болсон. 90%-ийн availability SLO-ийн 2 минутын time-based error budget нь 12 секунд бөгөөд 10 секундын downtime энэ budget дотор байсан. Гэхдээ request-based availability нь олон failed request болон payment error-оос шалтгаалан 90%-иас доош гарсан. Эцэст нь report-ийн p95 threshold-ийг 100 ms болгож зориудаар failure үүсгэхэд k6 `exit=99` буцаасан.

## 10. GitHub

Repository:

```text
https://github.com/AsilanAsKa/F.CSA313-Lab3
```

Branch:

```text
main
```
