Java (ayniqsa Spring Framework) da **@Transactional** — bu *transaction management* (tranzaksiyani boshqarish) uchun ishlatiladigan eng asosiy vositalardan biridir. Tranzaksiya — bu ma’lumotlar bazasidagi bir nechta amallarni bitta butun operatsiya sifatida bajarish demakdir: **hammasi muvaffaqiyatli bo‘lsa commit, xatolik bo‘lsa rollback.** Quyida to‘liq tushuntirish:

---

## 1️⃣ Tranzaksiya nima?

* **Atomicity** – Hamma ishlar bir butun bo‘lib bajariladi.
* **Consistency** – Bazadagi ma’lumotlar yaxlitligi saqlanadi.
* **Isolation** – Parallel ishlaydigan tranzaksiyalar bir-biriga aralashmaydi.
* **Durability** – Commit bo‘lgan ma’lumotlar saqlanib qoladi.

Masalan: bankdagi pul o‘tkazish (A hisobidan yechib, B hisobiga qo‘shish). Bir qismi muvaffaqiyatli, bir qismi xato bo‘lsa – hammasi bekor qilinadi.

---

## 2️⃣ Spring @Transactional nima qiladi?

Spring AOP (Aspect-Oriented Programming) orqali **Proxy** yaratiladi:

1. Metod chaqirilganda `TransactionManager` yangi tranzaksiya ochadi yoki mavjudiga qo‘shadi.
2. Metod normal tugasa → **commit**.
3. Exception (runtime/unchecked) chiqsa → **rollback** (yoki konfiguratsiyaga ko‘ra checked istisnolarni ham).

---

## 3️⃣ Qayerda ishlatiladi?

* **Service layer**: Biznes mantiq kodida (`@Service`).
* Ba’zida `@Repository` metodlarida, lekin odatda Service darajasida.

Misol:

```java
@Service
public class PaymentService {
    @Transactional
    public void transfer(Long from, Long to, BigDecimal amount) {
        accountRepo.debit(from, amount);
        accountRepo.credit(to, amount);
    }
}
```

---

## 4️⃣ Muhim parametrlar

| Parametr        | Tavsif                                           | Default                |
| --------------- | ------------------------------------------------ | ---------------------- |
| `propagation`   | Tranzaksiya chegarasini qanday kengaytirish      | REQUIRED               |
| `isolation`     | Boshqa tranzaksiyalar bilan izolyatsiya darajasi | DEFAULT (DBga bog‘liq) |
| `readOnly`      | Faqat o‘qish (optimallashtirish)                 | false                  |
| `timeout`       | Sekundlarda vaqt cheklovi                        | DB default             |
| `rollbackFor`   | Qaysi exception lar rollback qiladi              | RuntimeException       |
| `noRollbackFor` | Qaysi exception rollback qilmaydi                | —                      |

### Propagation turlari (asosiylari):

* **REQUIRED** – Mavjud bo‘lsa ulanish, bo‘lmasa yangi.
* **REQUIRES\_NEW** – Doim yangi tranzaksiya.
* **SUPPORTS** – Mavjud bo‘lsa ishlaydi, bo‘lmasa tranzaksiyasiz.
* **MANDATORY** – Mavjud bo‘lmasa xatolik.

### Isolation darajalari:

* **READ\_UNCOMMITTED** – “Dirty read”ga ruxsat.
* **READ\_COMMITTED** – Commit bo‘lgandan keyin o‘qiydi.
* **REPEATABLE\_READ** – O‘qigan satrlar commitgacha o‘zgarmaydi.
* **SERIALIZABLE** – Eng qat’iy, eng sekin.

---

## 5️⃣ Checked vs Unchecked Exception

* **Unchecked (RuntimeException, Error)** – avtomatik rollback.
* **Checked (Exception)** – rollback bo‘lishi uchun `rollbackFor=Exception.class` deb yozish kerak:

```java
@Transactional(rollbackFor = Exception.class)
public void process() throws Exception { ... }
```

---

## 6️⃣ Ichki ishlash mexanizmi

1. Spring konteyneri bean yaratganda tranzaksiyali metodni **proxy** bilan o‘rab qo‘yadi (JDK dynamic proxy yoki CGLIB).
2. Siz metodni chaqirganda, avval proxy ishga tushadi, `PlatformTransactionManager` orqali:

   * `begin()`
   * Metod ishlaydi
   * `commit()` yoki `rollback()`

⚠️ Shu sababli **self-invocation** (bir bean ichida o‘z metodini chaqirish) tranzaksiyani ishga tushirmaydi — proxy chetlab o‘tiladi.

---

## 7️⃣ Eng ko‘p uchraydigan xatolar

* **Self-invocation**: Bitta bean ichida `this.method()` chaqirilsa, tranzaksiya ishlamaydi.
* **Wrong propagation**: Noto‘g‘ri propagation ishlatish deadlockga olib kelishi mumkin.
* **LazyInitializationException**: Session yopilib ketganda lazy fieldlarga murojaat qilish.

---

## 8️⃣ Amaliy maslahatlar

* Service qatlamida e’lon qilish — yaxshi arxitektura.
* Faqat ma’lumot o‘qiladigan metodlar uchun `readOnly = true` qo‘llang.
* Uzoq davom etuvchi operatsiyalarni (masalan, tashqi API chaqiruvlari) tranzaksiya ichida qilmang.

---

### Qisqa xulosa

`@Transactional` – Springning kuchli tranzaksiya boshqaruv vositasi. U AOP orqali metodni proxy qiladi, avtomatik commit/rollback, propagation va isolation bilan ma’lumotlar yaxlitligini ta’minlaydi. To‘g‘ri qo‘llansa, kompleks ma’lumotlar bazasi amallarini ishonchli va sodda usulda bajarish imkonini beradi.
