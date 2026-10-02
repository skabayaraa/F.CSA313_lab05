# Лабораторийн ажил 05

- Оюутны нэр: Э.Энхсаруул
- Оюутны код: B232270145
- Node.js: v25.9.0
- Newman CLI: 6.2.2

## 1. Ажлын орчин ба тохиргоо

Лабын шаардлагын дагуу локал Node.js API-г ажиллуулж, түүнийгээ Postman collection-аар тохируулж, Newman CLI-ээс автоматаар шалгалаа. Бүх тестүүд `{{baseUrl}}`-ийг `http://localhost:3000` болгон ашиглаж, server.js-ийг локал орчинд 3000 портоор ажиллуулсан.

## 2. Сонголт ба төлөөлөх утгууд

| Сонголт | Тайлбар | Төлөөлөх утгууд |
|---|---|---|
| studentID-ийн хүчинтэй байдал | Оюутан бүртгэлтэй эсэх, идэвхтэй/идэвхгүй байдал | active, inactive, missing/unknown |
| Оюутны үзсэн хичээлүүд | Урьдчилсан prerequisite-ийг хангаж буй эсэх | [CS201], [], [CS101] |
| courseID-ийн хүчинтэй байдал | Хичээлийн мэдээлэл байгаа/байхгүй | CS313, CS999 |
| Хичээлийн prerequisite | Бүх урьдчилсан хичээлийг үзсэн, ганц нэгийг нь дутуу үзсэн, эсвэл огт байхгүй | [CS201], [], [CS201, CS202] |

## 3. Спецификаци ба хүлээгдэж буй үр дүн

| № | Спецификаци | Хүсэлт | Хүлээгдэж буй status | Хүлээгдэж буй result |
|---|---|---|---|---|
| 1 | Happy path: active student, prerequisite satisfied | POST /registrations | 201 | {"result":"OK"} |
| 2 | Оюутан байхгүй | POST /registrations | 200 | {"result":"ERROR_NO_STUDENT"} |
| 3 | Оюутан идэвхгүй | POST /registrations | 200 | {"result":"ERROR_INACTIVE_STUDENT"} |
| 4 | Хичээл байхгүй | POST /registrations | 200 | {"result":"ERROR_NO_COURSE"} |
| 5 | Урьдач нөхцөл дутуу | POST /registrations | 200 | {"result":"ERROR_PREREQUISITES","missing":["CS201"]} |
| 6 | studentID буюу courseID талбар дутуу | POST /registrations | 400 | {"result":"ERROR_BAD_REQUEST"} |
| 7 | Буруу JSON | POST /registrations | 400 | {"result":"ERROR_BAD_JSON"} |
| 8 | Давхар алдаа: inactive student + missing course | POST /registrations | 200 | {"result":"ERROR_INACTIVE_STUDENT"} |
| 9 | Урьдчилсан prerequisite байхгүй хичээл | POST /registrations | 201 | {"result":"OK"} |
| 10 | Empty coursesTaken + no prerequisites | POST /registrations | 201 | {"result":"OK"} |

## 4. Collection бүтэц

Collection дээр 10 бие даасан тестийг ашиглав. Тус бүрт setup PUT `/students/:id` болон PUT `/courses/:id`-ийг өөрийнхөө дагуу хийж, дараа нь POST `/registrations`-ийг явуулж, status болон result-ийг oracle-оор шалгав. registrationID-ийг яг утгаар нь шалгаагүй; зөвхөн property байгаа эсэхийг шалгасан.

- Үндсэн pass collection: `lab05-collection.json`
- Intentional fail collection: `lab05-collection-fail.json`

## 5. Newman ажиллуулалтын үр дүн

### 5.1 PASS

`newman run lab05-collection.json 2>&1 | tee results/newman-pass.txt`

- Хугацаа: 2026-10-02
- Assertions executed: 22
- Assertions failed: 0
- Exit code: 0
- Файл: `results/newman-pass.txt`

### 5.2 FAIL

`newman run lab05-collection-fail.json 2>&1 | tee results/newman-fail.txt`

- Assertions executed: 22
- Assertions failed: 1
- Exit code: 1
- Файл: `results/newman-fail.txt`

### 5.3 DOWN

`newman run lab05-collection.json 2>&1 | tee results/newman-down.txt`

- Серверийг унтраасны дараа `connect ECONNREFUSED 127.0.0.1:3000` гарсан.
- Энэ нь oracle-ийн алдаа биш, API рүү холбогдож чадаагүй интерфейсийн алдаа бөгөөд тиймээс `response` объект байхгүй, `pm.response.json()` ажиллахгүй. Exit code: 1.
- Файл: `results/newman-down.txt`

## 6. Нийтийн API нэмэлт гүнзгийрэл

`GET https://jsonplaceholder.typicode.com/users` рүү 3 oracle-ээр шалгасан:

1. HTTP status 200 байна.
2. Хариулт массив байна.
3. Эхний хэрэглэгчийн `name` нь `Leanne Graham` байна.

Энэ нь local API-тай харьцуулахад setup/cleanup шаардлагагүй, state-тэй ажиллахгүй, зүгээр л GET-only тул илүү энгийн боловч API-ны хэлбэрийн шалгалт, согогийн үзүүлэлтэд илүү төвлөрдөг.

## 7. Дүгнэлт

Лекцийн 5 алхмын хамгийн их бодол шаардсан хэсэг нь спецификацийн үе байсан, учир нь нэг хүсэлтэд хэд хэдэн боломжит алдаа байж болох тул эрэмбэ, precedence-ийг тодорхойлох шаардлагатай байсан. Хамгийн хэцүү хослол нь `inactive student + missing course` байсан бөгөөд сервер нь эхлээд оюутны статусаа шалгадаг тул энэ нь `ERROR_INACTIVE_STUDENT`-ээр буцсан. Тестүүдэд бид `registrationID`-ийн яг утгыг биш, зөвхөн `property` байгаа эсэхийг шалгасан нь server in-memory state-ээс болж давтан ажиллах үед тогтвортой байна. Нийтийн API-ийн GET шалгалт нь local API-тай харьцуулахад setup орхигдож, хялбар байсан ч хүссэн утга нь static байдгаараа онцлогтой байв. Бид пас, fail, down горим бүрийг ажиллуулж, true positive, intentional failure, болон connectivity error-ийг ялган салгасан нь QA-ийн үндсэн арга барилтай нийцсэн. Судалгааны явцад `ERROR_BAD_REQUEST` болон `ERROR_BAD_JSON` нь өөр өөр логик замаар гардаг болохыг баттай шалгасан. Төгсөгчийн хувьд API-ний хариу нь response code болон payload-ийг хоёуланг нь хянаж байх нь хамгийн найдвартай байдаг. Энэхүү лабораторийн ажлын үр дүнд automation pipeline нь логик алдаа, холболтын алдаа, чанарын хаалга зэрэг бүгдийг нэг дор илрүүлэх боломжтойг харууллаа.

## 8. Results файлд суурилсан товч хураангуй

- `results/newman-pass.txt`: assertions failed 0, exit code 0
- `results/newman-fail.txt`: assertions failed 1, exit code 1
- `results/newman-down.txt`: `connect ECONNREFUSED 127.0.0.1:3000`, exit code 1

