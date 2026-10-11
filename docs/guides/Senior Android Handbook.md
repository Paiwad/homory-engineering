# คัมภีร์ Senior Android Engineer

> คู่มือทบทวนเพื่อเปลี่ยนจาก “เขียนหน้าจอและเรียก API ได้” เป็น “ออกแบบระบบ Android ที่เชื่อถือได้ อธิบาย trade-off ได้ และแก้ปัญหาจากหลักฐานได้”
>
> ใช้คู่มือนี้เป็นแผนเรียน, checklist ก่อนเปิด PR และคลังคำตอบสัมภาษณ์ ไม่ใช่กฎตายตัว—ให้เลือกความซับซ้อนตาม product risk

**วิธีทบทวน:** อ่านทีละหัวข้อแล้วตอบ 3 คำถาม: “ปัญหาอะไร”, “เลือกวิธีนี้เมื่อไร”, “จะพิสูจน์ว่าถูกได้อย่างไร” ตัวอย่างในเล่มใช้หน้าขายสินค้าและการชำระเงินเป็นบริบทเดียวกัน เพื่อเห็นว่าหลักการแต่ละข้อเชื่อมกันอย่างไร

---

## แผนที่ความสามารถ

Senior ไม่ได้หมายถึงรู้ทุก library แต่หมายถึงทำสิ่งต่อไปนี้ได้อย่างสม่ำเสมอ:

1. แปลง product requirement ที่คลุมเครือเป็น state, invariant และ failure mode ที่ตรวจสอบได้
2. เลือก boundary และ dependency direction ที่ทำให้เปลี่ยนแปลงได้โดยไม่พังทั้งแอป
3. ทำให้ข้อมูลและ transaction ถูกต้องภายใต้ lifecycle, network failure, retry และ concurrent event
4. วัดก่อนแก้ performance/crash และพิสูจน์ผลหลังแก้
5. ถ่ายทอดเหตุผล, trade-off และความเสี่ยงให้ทีมตัดสินใจร่วมกันได้

ลำดับการเรียนที่คุ้มที่สุด:

```text
Kotlin + coroutine
  → Android lifecycle + UI state
  → data consistency + offline
  → architecture + modularization
  → testing + observability + performance/security
  → system/product design + leadership
```

---

## 1. Kotlin ที่ต้องเข้าใจลึก

### Nullability และ type system

- `String?` หมายถึง “มีข้อความหรือไม่มีค่า” ตามที่ type อนุญาต เช่น `middleName: String?` เพราะลูกค้าอาจไม่มีชื่อกลางจริง ๆ ผู้เรียกจึงต้องจัดการ `null` ไม่ใช่แค่ป้องกัน NPE
- ถ้า `null` อาจแปลว่า “ยังไม่โหลด”, “ไม่พบ”, หรือ “โหลดล้มเหลว” ให้แยก state ด้วย type: แต่ละกรณีต้องแสดง UI หรือ retry ต่างกัน การยัดทุกความหมายใน `null` ทำให้ caller เดาผิดได้
- หลีกเลี่ยง `!!` ใน production; แปลง input ที่ไม่เชื่อถือได้เป็น validated type ที่ boundary
- ใช้ `sealed interface`/`sealed class` แทน `Boolean` หรือ nullable หลายตัว เมื่อ state มีความหมายต่างกัน: `when` จะตรวจว่าจัดการทุกกรณีครบหรือไม่
- เลือก `sealed class` เมื่อ subtype ต้องใช้ state/implementation ร่วมจาก base class; เลือก `sealed interface` เมื่ออยากกำหนดเพียง contract และให้ class implement interface อื่นได้ด้วย
- `data class` เหมาะกับ value; อย่าใช้ equality ของ object เป็น business identity โดยไม่ตั้งใจ
- เข้าใจ variance: `out` ผลิตค่า, `in` รับค่า; มักพบใน generic API และ collection

```kotlin
sealed interface PaymentStatus {
    data object Idle : PaymentStatus
    data class Processing(val transactionId: String) : PaymentStatus
    data class Completed(val receiptNo: String) : PaymentStatus
    data class Failed(val reason: PaymentFailure) : PaymentStatus
}
```

ตัวอย่าง `PaymentStatus.Failed` มี `reason` แต่ `Completed` มี `receiptNo` จึงไม่ต้องสร้าง object ที่มี `isSuccess: Boolean`, `receiptNo: String?`, `error: String?` ซึ่งเปิดทางให้ state ขัดกัน เช่น สำเร็จแต่มี error พร้อมกัน

### Immutability และ side effect

- ส่ง immutable model ระหว่าง layer; mutation ที่กระจัดกระจายทำให้ state race และ test ยาก
- `copy()` เป็น shallow copy: collection/object ข้างในยังอาจ mutable ได้
- แยก pure function (input → output) ออกจาก I/O เช่น network, database, clock, UUID
- inject `Clock`, ID generator, dispatcher หรือ gateway เมื่อสิ่งนั้นทำให้ผลลัพธ์ไม่ deterministic ใน test

ตัวอย่าง: `state.copy(items = state.items + newItem)` สร้าง list ใหม่ แต่ถ้า `CartItem` ข้างในมี `var price` การแก้ `price` ยังเปลี่ยนข้อมูลที่ state เก่าอ้างอยู่ได้ จึงควรใช้ immutable item ด้วย

### Scope functions และ extension

- ใช้ `let` สำหรับ transform nullable, `also` สำหรับ side effect, `apply` ตั้งค่า object, `run/with` รวม expression
- ถ้าอ่านแล้วไม่ชัด เขียนตัวแปรปกติดีกว่า chain ยาว ๆ
- extension ไม่ใช่ polymorphism; ถูก resolve แบบ static และไม่ควรซ่อนงานหนัก/side effect ที่คนเรียกคาดไม่ถึง

### Error model

- Exception เหมาะกับ failure ที่ข้ามหลายชั้นหรือเกิดจาก library; expected business outcome ควรเป็น typed result
- อย่าใช้ `Result` เป็น universal wrapper ทุก layer โดยไม่รู้ว่าใครแปลง error ตรงไหน
- แยก error ที่ user แก้ได้ (`Validation`, `Unauthorized`, `OutOfStock`) ออกจาก technical failure (`Timeout`, `Unknown`)

---

## 2. Coroutine และ Flow

### Structured concurrency

- งานต้องมี owner: UI event มักอยู่ใน `viewModelScope`; งานที่ต้องทำต่อแม้ UI หายให้ใช้ `WorkManager` หรือ application-owned scope ตาม use case
- อย่าสร้าง `CoroutineScope()` หรือ `GlobalScope` เองเพื่อหลบ cancellation—มักทำให้ leak และ error หาย
- `coroutineScope`: child fail หนึ่งตัว cancel sibling ทั้งหมด เหมาะเมื่อผลลัพธ์ต้องสำเร็จครบ
- `supervisorScope`: child fail ไม่ cancel sibling เหมาะกับข้อมูลอิสระ แต่ต้อง handle exception ของแต่ละงานเอง
- `async` ใช้เมื่อจะ combine ผลลัพธ์; `launch` ใช้เมื่องานไม่มีผลลัพธ์กลับมา

ตัวอย่าง: โหลดราคาและส่วนลดเพื่อคำนวณยอดเดียวกัน ใช้ `coroutineScope` เพราะขาดอย่างใดอย่างหนึ่งคำนวณไม่ได้; โหลด banner และรายการสินค้าอิสระต่อกัน ใช้ `supervisorScope` ได้ถ้า UI แสดงส่วนที่สำเร็จและจัดการ failure ของส่วนที่ล้มเหลวแยกกัน

### Dispatcher

- Main: update UI/state; ห้ามทำ I/O หรือ CPU-heavy
- IO: blocking disk/network API
- Default: CPU-heavy เช่น parse, encrypt, filter รายการใหญ่
- Suspend Retrofit/Room ส่วนมากจัด dispatcher ของตัวเองได้; อย่าครอบ `withContext(IO)` ซ้ำอย่างไร้เหตุผล
- Dispatcher ควรถูก inject สำหรับ test หาก code สลับ context เอง

### Cancellation

- cancellation เป็น flow ปกติ ไม่ควรถูกแปลงเป็น `Error` ให้ผู้ใช้
- อย่ากลืน `CancellationException` ด้วย `catch (Exception)` หรือ `runCatching` แบบกว้าง ๆ
- library/network operation ต้อง cooperative cancellation จึงจะหยุดจริง

เช่น ผู้ใช้พิมพ์คำค้นใหม่ `flatMapLatest` ยกเลิกงานค้นเดิม การยกเลิกนี้ไม่ใช่ “ค้นหาล้มเหลว” ถ้าต้องจับ `Exception` ให้ปล่อย `CancellationException` ผ่านก่อนแปลง error อื่นเป็น UI state

### Flow เลือกให้ตรงความหมาย

| สิ่งที่ส่ง | เครื่องมือ | คุณสมบัติสำคัญ |
|---|---|---|
| state ล่าสุดของหน้าจอ | `StateFlow` | collector ใหม่ได้ latest value ทันที |
| event กระจายถึงหลาย listener | `SharedFlow` | กำหนด replay/buffer ได้ |
| queue ที่หนึ่ง event ให้หนึ่ง consumer | `Channel` + `receiveAsFlow()` | ownership ของ event ชัด |
| ข้อมูลตามเวลา/หลายค่า | `Flow` | cold stream, เริ่มเมื่อ collect |

- ข้อมูลที่ต้องกลับมาหลัง rotate ให้เป็น state ไม่ใช่ one-off event
- navigation/toast เป็น transient event ได้ แต่ต้องคิดว่าหาก collector ไม่ active จะ drop, buffer หรือ encode เป็น state อย่างไร
- UI collect ด้วย `repeatOnLifecycle(Lifecycle.State.STARTED)` ไม่ใช่ collect ตลอดชีวิต Fragment

จำง่าย: ยอดเงินในตะกร้าคือ **state** เพราะจอใหม่ต้องเห็นค่าเดิม; คำสั่งเปิดหน้าใบเสร็จคือ **effect** เพราะไม่ควรเปิดซ้ำทุกครั้งที่ collect ใหม่ แต่ถ้าใบเสร็จต้องกู้คืนหลัง process death ให้เก็บ `receiptId` เป็น state แล้วออกแบบการนำทางจาก state นั้นอย่างชัดเจน

### Search ที่ไม่ stale

```kotlin
query
    .map(String::trim)
    .debounce(300)
    .distinctUntilChanged()
    .flatMapLatest(repository::search)
```

`flatMapLatest` cancel query เก่าเมื่อมี keyword ใหม่ จึงไม่ให้ผลเก่ามาทับผลใหม่; งาน filter in-memory ขนาดใหญ่ให้ย้ายไป `Default`, database ให้มี index, list ยาวให้พิจารณา Paging

---

## 3. Lifecycle, UI State และ Compose/View

### หลักคิด state

UI คือ function ของ state: หน้าจอควร render ได้จาก `UiState` โดยไม่พึ่งลำดับ callback ก่อนหน้า

```kotlin
data class CheckoutUiState(
    val items: List<CartItem> = emptyList(),
    val isSubmitting: Boolean = false,
    val error: CheckoutError? = null,
)
```

ลองถามว่า `isSubmitting = true` และ `error != null` พร้อมกันมีความหมายไหม ถ้าไม่ ให้ใช้ sealed state เช่น `Editing`, `Submitting`, `Success`, `Failure` เพื่อห้ามชุดค่าที่ขัดกันตั้งแต่ type level

- อย่า reset เป็น `Idle` ทันทีหลัง success ถ้า UI ที่ recreate ต้องรู้ผลล่าสุด
- แยก persistent screen state ออกจาก transient effect
- แทน flag หลายตัว (`isLoading`, `hasError`, `isEmpty`) ด้วย sealed state เมื่อ combination บางแบบ invalid
- Event จาก UI เป็น intent/action ที่มีความหมาย เช่น `RetryClicked`, `PaymentConfirmed`; ViewModel เป็นคนลด action เป็น state

### ViewModel และ process death

- `ViewModel` รอด configuration change แต่ **ไม่รอด process death**
- `SavedStateHandle` เก็บเฉพาะ key เล็ก ๆ ที่ใช้กลับสู่จุดเดิม เช่น selected ID, query—not object/list ใหญ่หรือ secret
- ข้อมูล source of truth อยู่ database/server; restore จาก ID แล้วโหลดใหม่
- Fragment/Activity ไม่ควรถือ business state หรือเริ่ม request ที่ถูกยิงซ้ำทุก recreation

ภาพจำ: หมุนจอคือสร้าง Activity ใหม่แต่ `ViewModel` เดิมมักยังอยู่; process death คือทั้ง process และ `ViewModel` หาย การกลับมาหน้าชำระเงินจึงควรใช้ `transactionId` ที่เก็บไว้ตรวจสถานะจากแหล่งข้อมูลจริง ไม่เดาจาก flag ในหน่วยความจำ

### Compose ที่มักพลาด

- composition ไม่ใช่ lifecycle ของ screen; recomposition เกิดได้บ่อยมาก
- อย่าทำ network/write/analytics ตรง body ของ composable; ใช้ event handler, `LaunchedEffect` หรือ side-effect API ที่เหมาะสม
- `remember` อยู่ใน composition, `rememberSaveable` เก็บ UI value เล็ก ๆ, ViewModel เก็บ screen/business state
- `LaunchedEffect(key)` จะ restart เมื่อ key เปลี่ยน; key ผิดทำให้ call API ซ้ำหรืองานไม่ restart
- State ที่ส่งเข้าลูกควร stable/immutable เพื่อให้ recomposition คาดเดาได้
- list ใช้ stable key: `items(items, key = { it.id })`

ตัวอย่าง `LaunchedEffect(Unit)` ทำงานเมื่อ composable เข้าสู่ composition ใหม่ ไม่ใช่คำสัญญาว่าจะทำครั้งเดียวตลอดอายุธุรกรรม งานชำระเงินควรเริ่มจาก user action ที่ ViewModel ควบคุมและมี idempotency key

### XML/View ที่ยังต้องรู้

- clear ViewBinding ใน `Fragment.onDestroyView()` ไม่ใช่ `onDestroy()`
- อย่าเก็บ `Activity`/`View` reference ใน ViewModel, singleton หรือ callback ระยะยาว
- `RecyclerView`: `ListAdapter` + `DiffUtil`, stable ID เมื่อเหมาะสม, อย่า `notifyDataSetChanged()` ทุกครั้ง

---

## 4. Architecture และ Modularization

### Clean Architecture แบบใช้งานได้จริง

```text
presentation → domain ← data
       ↓                 ↓
 Android/UI APIs     Retrofit / Room / file / device
```

- domain มี business rule, entity/use case และ interface ที่ business ต้องการ; ไม่ import Android/Retrofit/Room
- data implement interface และ map DTO/entity ↔ domain model
- presentation ทำ UI state, event และ orchestration ที่เป็น UI-specific
- ไม่จำเป็นต้องสร้าง use case 1:1 ทุก repository method; สร้างเมื่อรวม business rule, transaction, permission หรือเป็น stable business boundary

### Dependency inversion ที่ถูกที่

```kotlin
interface PaymentRepository {
    suspend fun submit(command: PaymentCommand): PaymentResult
}
```

Domain เป็นเจ้าของ abstraction เมื่อ domain ต้องการความสามารถนั้น—not data เป็นคนกำหนด API ตาม Retrofit response

อ่านลูกศรในแผนภาพเป็น “module นี้รู้จัก type ของ module ไหน”: presentation และ data อ้าง domain; domain ไม่ต้องรู้จัก Retrofit, Room หรือ Android UI ตอน runtime DI นำ implementation ของ data มาผูกกับ interface ที่ domain ใช้

### Module เมื่อมีเหตุผล

- เริ่มจาก `:app`, `:core:domain`, `:core:data`, `:feature:home` ได้
- `:core:designsystem` เมื่อหลาย feature ใช้ theme/component เดียวกันจริง
- pure Kotlin: **Java or Kotlin Library**
- ใช้ Android Library เฉพาะ module ที่ต้อง import Android SDK/resource/Compose Android
- อย่าแตก module เพียงเพราะ package เยอะ: module มี build/configuration cost และ dependency graph ที่ต้องดูแล
- หลีกเลี่ยง `:core` ก้อนใหญ่; ตั้งชื่อตาม responsibility เช่น `:core:network`, `:core:database`, `:core:ui`

### DI

- constructor injection เป็น default; dependency ชัดและ unit test ง่าย
- Hilt/Dagger module ใช้ bind/provide ที่ infrastructure boundary
- อย่า inject `Context` เข้า domain/ViewModel ถ้าไม่จำเป็น; แปลง Android concern เป็น abstraction
- Scope ให้ตรง lifetime: singleton เฉพาะ stateless/long-lived resource ที่ thread-safe, ViewModel scope สำหรับ screen state

---

## 5. Data, Offline-first และ Consistency

### Source of truth

- UI ควรอ่านจาก local database เมื่อ product ต้องรองรับ offline/consistency; network sync มา update database แล้ว UI observe
- อย่าส่ง API response ตรงไป UI และ database แบบแยกกันโดยไม่กำหนดว่าอะไรคือ source of truth
- แยก model: Network DTO, database entity, domain model, UI model เมื่อ boundary ต่างกันจริง

ลำดับตัวอย่าง: ผู้ใช้เพิ่มสินค้า, repository บันทึก local DB, UI observe แถวที่เปลี่ยน, sync ส่งไป server แล้วบันทึกผลกลับ DB วิธีนี้เหมาะเมื่อข้อกำหนดต้องให้ข้อมูลยังอยู่หลังปิดแอปหรือ offline; ถ้าไม่มีข้อกำหนดนี้ ไม่จำเป็นต้องเพิ่ม DB ทุกหน้าจอ

### Sync ต้องตอบคำถามให้ได้

1. ใครเป็น source of truth เมื่อ online/offline?
2. งานที่ค้างถูกเก็บที่ไหน และ survive app kill ไหม?
3. retry เมื่อไร, backoff อย่างไร, user เห็นอะไร?
4. idempotency key คืออะไร?
5. conflict เกิดได้อย่างไร และใช้ last-write-wins/merge/manual resolution แบบใด?
6. การลบ sync อย่างไร? (`deletedAt` tombstone มักจำเป็น)

### Transaction สำคัญ: payment, order, stock

- client-side debounce เป็น UX guard เท่านั้น ไม่ใช่ data integrity
- สร้าง `transactionId`/idempotency key หนึ่งครั้งต่อ intent การชำระ และเก็บให้ recover ได้
- API/server ต้อง enforce uniqueness atomically; duplicate success ควรคืนผลเดิมหรือสถานะเดิม
- timeout หมายถึง “ไม่รู้ผล” ไม่ใช่ “ล้มเหลว”; query status ด้วย transaction ID ก่อนสร้างรายการใหม่
- state ที่ชัด: `Idle → Processing → Completed | RequiresStatusCheck | Failed`
- log correlation ID, transaction ID และ error class แต่ห้าม log secret/card/PII โดยไม่ redaction

ตัวอย่าง: กดจ่ายครั้งเดียวด้วย key `pay-123`; server ตัดเงินสำเร็จแต่ response หาย หาก retry ด้วย key เดิม server คืนผลของรายการเดิม หากสร้าง key ใหม่อาจเกิดการจ่ายซ้ำ จึงต้องเก็บ key จนรู้ผลแน่ชัด และให้ server บังคับ uniqueness จริง

### Room และ database

- ทำ migration test ทุก schema migration; destructive migration สำหรับ production data ต้องมีเหตุผลชัด
- ใส่ index ให้ column ที่ search/filter/join บ่อย และดู query plan เมื่อข้อมูลโต
- group write ที่ต้อง all-or-nothing ด้วย transaction
- อย่า query list ใหญ่ทั้งก้อนบน Main; ใช้ Flow/Paging และ pagination

---

## 6. Network และ Error Handling

### แยก failure ให้ถูก

| ประเภท | ตัวอย่าง | แนวทาง UX |
|---|---|---|
| validation | form ไม่ครบ | บอก field ที่แก้ได้ทันที |
| authentication | token หมดอายุ | refresh/login ตาม policy |
| authorization | ไม่มีสิทธิ์ | บอกว่าทำไม่ได้ ไม่ retry รัว |
| connectivity | offline/DNS | offline UI, retry เมื่อเหมาะ |
| timeout | ผลลัพธ์ไม่แน่ชัด | query status สำหรับ mutation สำคัญ |
| server/business | stock ไม่พอ | แสดง business message ที่ปลอดภัย |

- retry เฉพาะ operation ที่ idempotent หรือมี idempotency key
- exponential backoff + jitter สำหรับ retry อัตโนมัติ; อย่า retry 4xx validation/permission
- timeout ของแต่ละชั้นต้องมี budget; ไม่ใช่ตั้ง client มากกว่า server เสมอ เพราะ gateway/load balancer อาจหมดก่อน
- interceptors ต้องไม่ทำ recursion และ token refresh ต้องกัน concurrent refresh storm

`GET` ที่อ่านข้อมูลมัก retry ได้ตาม policy; `POST /payments` retry ได้อย่างปลอดภัยก็ต่อเมื่อสัญญา API รองรับ idempotency key และ server enforce จริง สำหรับ timeout ของ payment ให้ตรวจสถานะเดิมก่อนบอกผู้ใช้ว่าจ่ายไม่สำเร็จ

---

## 7. Testing ที่ควรทำเป็นนิสัย

### Test pyramid ที่มีความหมาย

- unit test: pure business logic, mapping, validation, ViewModel reducer
- integration test: repository + Room/network fake, migration, serialization, DI wiring ที่เสี่ยง
- UI test: critical user journey, navigation, accessibility—not ทุก pixel/detail
- screenshot/golden test เมื่อ visual regression มีต้นทุนสูง

### สิ่งที่ต้อง test ใน coroutine/Flow

- loading → content/error transition
- cancellation และ request เก่าไม่ overwrite state ใหม่
- retry/idempotency/duplicate action
- dispatcher ไม่รั่วไป Main test
- virtual time ด้วย `runTest`, ใช้ test dispatcher ที่ควบคุมได้

### คุณภาพ test

- test behavior ไม่ test implementation detail
- ตั้งชื่อ `given_when_then` หรือชื่อ scenario ที่อ่านแล้วรู้ business rule
- fake เหมาะกับ behavior ที่ใช้ซ้ำ; mock เหมาะ verify collaboration เฉพาะจุด
- flaky test คือ defect: ห้ามแก้ด้วย `delay()` แบบเดาสุ่ม

ตัวอย่าง test ที่มีค่า: เมื่อส่ง payment แล้ว timeout, ViewModel แสดง “กำลังตรวจสอบผล” และการกด retry ไม่สร้าง `transactionId` ใหม่ Test นี้พิสูจน์ business invariant มากกว่าตรวจว่าเรียก private method กี่ครั้ง

---

## 8. Performance และ Reliability

### วัดก่อนแก้

- ใช้ Android Studio Profiler, Perfetto, Macrobenchmark, Baseline Profiles, Play Console/Crashlytics ตามปัญหา
- วัด startup, frame time/jank, memory, ANR, network, battery—not “รู้สึกว่าช้า”
- กำหนด metric/target ก่อน optimization เช่น cold start p95, scroll jank %, crash-free users

### สาเหตุยอดนิยม

- main-thread disk/network/JSON parse/filter ใหญ่
- nested layout หรือ Compose recomposition มากเกิน
- bitmap ใหญ่, image cache ผิด, memory leak
- list rebind ทุก item, ไม่มี paging/diff
- งาน background ที่ retry loop หรือ wake device บ่อย

### ANR และ crash

- ANR เป็นการ block main thread หรือ component lifecycle ที่ใช้เวลานาน; อ่าน trace ก่อนตัดสิน
- crash report ต้อง triage ด้วย version, device, breadcrumb, frequency และ impact
- fix ที่ดีต้องมี reproduction/test/metric หรือเหตุผลว่าทดสอบไม่ได้อย่างไร

---

## 9. Security และ Privacy

- threat model ก่อนเลือก encryption: ป้องกันใคร, asset อะไร, attack surface ไหน
- อย่าเก็บ token/password/PII ใน plain text, logs, screenshots หรือ analytics event
- ใช้ Android Keystore ผ่าน API ที่เหมาะสม; encryption ไม่ชดเชย authorization ฝั่ง server
- certificate pinning มี operational cost (rotation/outage); ใช้เมื่อ risk model รองรับ
- validate deep link/input ทุก boundary; อย่าเชื่อ Intent extra, QR payload หรือ server string โดยไม่มี validation
- ลด permission และ data collection ตาม least privilege
- release build: ปิด debug behavior, review exported component, backup policy, network security config และ secret management

---

## 10. Accessibility และ UX ของ failure

- ทุก control ต้องมี content description/label ที่ meaningful และ touch target เพียงพอ
- อย่าพึ่งสีอย่างเดียวเพื่อสื่อ error/success; รองรับ font scaling, TalkBack, contrast และ RTL หาก target ต้องรองรับ
- loading ต้องไม่ทำให้ผู้ใช้กด action อันตรายซ้ำ และควรบอกสถานะที่กำลังเกิด
- empty, loading, error, offline, permission denied คือ state ของ product—not edge case ที่ค่อยทำท้ายสุด
- destructive action ต้อง confirmation/recovery ที่เหมาะกับ impact

---

## 11. การ Debug แบบ Senior

### Loop

1. เก็บข้อเท็จจริง: version, device, time, input, logs, trace, network/state
2. ทำ reproduction ที่เล็กที่สุด
3. เขียน hypothesis ที่ falsifiable
4. trace data flow และ error flow จาก entry point ถึง side effect
5. แก้ root cause พร้อม test/guardrail
6. ตรวจผลด้วย test, metric หรือ reproduction เดิม

### คำถามที่ต้องถามเสมอ

- state นี้เปลี่ยนที่ไหนบ้าง? ใครเป็น owner?
- หาก rotate, background, process death, offline, retry หรือ callback มาช้า จะเกิดอะไร?
- operation นี้ idempotent ไหม? duplicate ได้จากทางใด?
- อะไรคือ source of truth?
- error นี้ควรเห็นโดย user, retry อัตโนมัติ หรือ report อย่างเดียว?
- มี evidence ว่าปัญหาหายแล้วหรือยัง?

---

## 12. การออกแบบและสื่อสาร

### ก่อน implement

เขียนสั้น ๆ ให้ได้:

```text
Goal: ผู้ใช้ทำอะไรสำเร็จ
Non-goal: อะไรที่ยังไม่ทำ
State: normal/loading/empty/error/edge states
Invariant: สิ่งที่ห้ามผิด เช่น charge เดียวต่อ transactionId
Failure: network, permission, server, concurrency, process death
Data ownership: source of truth และ sync direction
Verification: test/metric ที่ยืนยันผล
```

### ใน code review

- review correctness, lifecycle, concurrency, security, test และ maintainability ก่อน style
- comment ต้องระบุ location, impact และแนวทาง—not เพียง “ควร refactor”
- แยก blocking issue ออกจาก suggestion
- ไม่ขอ abstraction เพื่ออนาคตที่ยังไม่มี use case

### Trade-off ที่ตอบสัมภาษณ์ได้ดี

โครงคำตอบ: **Context → options → decision → downside → mitigation → validation**

ตัวอย่าง: “เลือก local DB เป็น source of truth เพราะหน้าขายต้องใช้ได้ offline; แลกกับ sync/conflict ที่ซับซ้อนขึ้น จึงมี outbox, idempotency key และ test กรณี retry/timeout; วัด sync success rate และรายการค้างเป็น metric”

**คำศัพท์ที่ใช้บ่อย:** `invariant` คือเงื่อนไขที่ห้ามผิด เช่น หนึ่ง `transactionId` ต้องไม่ถูกคิดเงินสองครั้ง; `source of truth` คือแหล่งข้อมูลที่ใช้ตัดสินค่าปัจจุบันเมื่อหลายแหล่งไม่ตรงกัน; `idempotency` คือทำคำขอซ้ำแล้วผลทางธุรกิจยังเท่ากับทำครั้งเดียว; `outbox` คือบันทึกงานที่รอส่งไว้ใน storage เพื่อส่งต่อและ retry ภายหลัง

---

## 13. Senior Android Interview Drill

ตอบแต่ละข้อโดยเริ่มจาก assumption แล้วให้ state/failure/trade-off:

1. หมุนจอระหว่าง API call แล้ว UI ต้องคง state อย่างไร?
2. `StateFlow`, `SharedFlow`, `Channel` ต่างกันอย่างไร และ event หายได้เมื่อไร?
3. `coroutineScope` และ `supervisorScope` เหมาะกับข้อมูลแบบใด?
4. จะป้องกัน payment ซ้ำจาก double tap, timeout และ retry อย่างไร?
5. ออกแบบ offline-first sync, conflict และ delete อย่างไร?
6. Search 5,000 รายการกระตุกและผลเก่าทับผลใหม่ แก้อย่างไร?
7. `ViewModel` รอดอะไรบ้าง และ process death recover อย่างไร?
8. จะแก้ ANR/crash ที่ยัง reproduce ไม่ได้อย่างไร?
9. จะ modularize แอปเล็กโดยไม่ over-engineer อย่างไร?
10. จะ test ViewModel ที่มี Flow, debounce และ retry อย่างไร?

---

## 14. Checklist ก่อนเปิด Pull Request

### Correctness

- [ ] ระบุ happy path, empty, loading, error, retry, offline และ permission state แล้ว
- [ ] ไม่มี duplicate side effect จาก recomposition, rotation หรือ repeated click
- [ ] mutation สำคัญมี idempotency/recovery path ตามความเสี่ยง
- [ ] data/source of truth และ mapping ระหว่าง layer ชัดเจน

### Lifecycle และ concurrency

- [ ] coroutine มี owner และ cancellation ถูกต้อง
- [ ] ไม่ update View หลัง lifecycle หยุด
- [ ] request เก่าไม่สามารถ overwrite state ใหม่
- [ ] ไม่กลืน `CancellationException`

### Quality

- [ ] test business/error/concurrency behavior ที่เปลี่ยนแล้ว
- [ ] ไม่มี main-thread blocking หรือ query/list ใหญ่โดยไม่จำเป็น
- [ ] log/analytics ไม่รั่ว PII/secret
- [ ] accessibility, font scale และ error copy ผ่านการพิจารณา
- [ ] รัน test/build/lint ที่เกี่ยวข้องและบันทึกผลจริง

---

## แผนฝึก 8 สัปดาห์

| สัปดาห์ | หัวข้อ | งานฝึกที่พิสูจน์ได้ |
|---|---|---|
| 1 | Kotlin, error model, immutability | เขียน validation/reducer พร้อม unit test |
| 2 | Coroutines/Flow | Search debounce + `flatMapLatest` + test stale result |
| 3 | Lifecycle/Compose | หน้าจอ survive rotate/process recreation ตามระดับที่กำหนด |
| 4 | Architecture/DI | แยก domain-data-presentation อย่างพอดีและอธิบาย dependency graph |
| 5 | Room/offline | local source of truth, migration test, paging/search index |
| 6 | Sync/transaction | outbox + retry + idempotency/status recovery simulation |
| 7 | Performance/reliability | ทำ baseline metric แล้วแก้ jank/startup อย่างน้อยหนึ่งปัญหา |
| 8 | Security/testing/interview | threat review, critical UI journey test, mock interview 10 ข้อ |

## แหล่งอ้างอิงที่ควรอ่านต่อ

- Android Developers: Architecture, Lifecycle, Coroutines, Compose, Room, WorkManager, Performance และ Security
- Kotlin documentation: Coroutines, Flow, cancellation และ type system
- Android Now in Android sample: modularization, offline-first, testing และ Compose conventions
- Google Android Performance Patterns / Perfetto documentation สำหรับการวัดจริง

> เป้าหมายของการ drill down ไม่ใช่ท่อง API: ทุกหัวข้อควรทำเป็น sample เล็ก ๆ, ใส่ failure case ตั้งใจ, เขียน test แล้วอธิบายได้ว่าเหตุใด design นี้จึงยังถูกต้องเมื่อ lifecycle และ network ไม่เป็นใจ
