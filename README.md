# F.CSA313 Lab 3

**Оюутан:** Асилан Серикбай
**Оюутны код:** B232270115
**k6 version:** v2.2.0

## 1. Лабораторийн зорилго

Энэ лабораториор Express API дээр k6 ашиглаж performance, reliability, availability тест хийсэн. Мөн SLO, threshold, error budget болон chaos test хийж үзсэн.

## 2. API

Энэ лабораторид Node.js + Express ашигласан.

Үндсэн API:

* `POST /cart/add` – cart-д item нэмнэ.
* `GET /report` – report буцаана. 200–400 ms орчим delay-тэй.
* `POST /pay` – payment хийж, ойролцоогоор 5% үед 500 error өгнө.

Server:

```text
http://localhost:3000
```

## 3. Quality Scenarios

### Performance

| Зүйл        | Тайлбар                                   |
| ----------- | ----------------------------------------- |
| Source      | 20 VU хэрэглэгч                           |
| Stimulus    | `/cart/add`, `/report` рүү request илгээх |
| Environment | Local Express server                      |
| Artifact    | API response                              |
| Metric      | Response time p95                         |
| Response    | Threshold шалгах                          |

### Reliability

| Зүйл        | Тайлбар                             |
| ----------- | ----------------------------------- |
| Source      | Хэрэглэгч                           |
| Stimulus    | `/pay` рүү олон request илгээх      |
| Environment | Local Express server, 20 VU         |
| Artifact    | Payment response                    |
| Metric      | Error rate, response time p95       |
| Response    | Payment error rate threshold шалгах |

### Availability

| Зүйл        | Тайлбар                                            |
| ----------- | -------------------------------------------------- |
| Source      | Хэрэглэгч                                          |
| Stimulus    | 2 минут test хийх үед server-ийг 10 секунд зогсоох |
| Environment | Local Express server, 20 VU                        |
| Artifact    | API response                                       |
| Metric      | Successful checks, availability                    |
| Response    | SLO болон error budget-тэй харьцуулах              |

## 4. SLO

Миний `slo-test.js` дээр дараах threshold тохируулсан.

| Metric         | Threshold |
| -------------- | --------- |
| Cart p95       | `<200ms`  |
| Report p95     | `<450ms`  |
| Pay error rate | `<8%`     |
| Checks         | `>90%`    |

### Error Budget

Availability SLO нь 90%.

2 минут = 120 секунд.

```text
120 × (1 - 0.90) = 12 секунд
```

Тиймээс 2 минутын test үед 12 секундийн downtime нь error budget болно.

## 5. Pass Test

`k6 run slo-test.js` ашиглан baseline test хийсэн.

Үр дүн:

| Metric         |    Result |
| -------------- | --------: |
| Checks         |    98.31% |
| Cart p95       |   1.88 ms |
| Report p95     | 391.15 ms |
| Pay error rate |     5.04% |
| HTTP failed    |     1.68% |
| Requests       |      2796 |
| Iterations     |       932 |

Энэ test дээр тохируулсан threshold-үүд pass болсон.

Бүрэн output:

`results/pass.txt`

## 6. Chaos Test

Chaos test хийхдээ server-ийг 10 секунд зогсоосон. Test нийт 2 минут ажилласан.

Үр дүн:

| Metric         |    Result |
| -------------- | --------: |
| Checks         |    84.28% |
| Cart p95       |   1.95 ms |
| Report p95     | 388.67 ms |
| Pay error rate |    18.45% |
| HTTP failed    |    15.71% |
| Requests       |      5739 |
| Iterations     |      1913 |

Checks 84.28% болсон учраас `>90%` threshold fail болсон.

Pay error rate 18.45% болсон учраас `<8%` threshold мөн fail болсон.

Харин:

* Cart p95 = 1.95 ms → `<200ms`
* Report p95 = 388.67 ms → `<450ms`

болсон тул performance threshold-үүд pass болсон.

### Error Budget

2 минутын SLO 90% үед:

```text
Error budget = 12 секунд
```

Server-ийг 10 секунд зогсоосон тул time-based downtime нь 12 секундын budget дотор байна.

Гэхдээ request-based availability 84.28% болсон учраас 90%-ийн SLO-д хүрээгүй.

Бүрэн output:

`results/chaos.txt`

## 7. Intentional Fail Test

Threshold failure шалгахын тулд `slo-test-fail.js` гэсэн тусдаа файл хийсэн.

Энэ файлд report-ийн threshold-ийг:

```text
p(95)<100ms
```

гэж зориуд өөрчилсөн.

Гэтэл бодит report p95:

```text
390.55 ms
```

байсан.

Тиймээс threshold fail болсон.

Test-ийн төгсгөлд:

```text
thresholds on metrics 'http_req_duration{name:report}' have been crossed
exit=99
```

гарсан.

Бүрэн output:

`results/fail.txt`

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

`node_modules/`-ийг `.gitignore` хийсэн.

## 9. Дүгнэлт

Энэ лабораториор k6 ашиглан API-ийн performance, reliability, availability-г шалгасан. Эхний test дээр бүх threshold pass болсон. Chaos test хийх үед server-ийг 10 секунд зогсоосноор checks 84.28% болж, pay error rate 18.45% болсон. Энэ үед availability болон reliability threshold fail болсон. Харин cart болон report-ийн response time threshold хэвийн байсан. Мөн 12 секундийн error budget-тай харьцуулж үзэхэд server-ийн 10 секундийн downtime нь time-based budget дотор байсан. Сүүлд threshold-ийг зориудаар `p(95)<100ms` болгож failure test хийхэд k6 `exit=99` буцаасан. Ингэж threshold ажиллаж байгааг шалгасан.

## 10. GitHub

Repository:

`https://github.com/AsilanAsKa/F.CSA313-Lab3`

Branch:

`main`


