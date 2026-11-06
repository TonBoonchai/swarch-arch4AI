# Task 4: Deployment topologies and trade-offs

**Name / ชื่อ:** บุญชัย จินาพร

**Student ID / รหัสนิสิต:** 6632110421

This is **individual work**: each student completes and submits their own copy, in their own fork.

*งานนี้เป็น**งานเดี่ยว** นิสิตแต่ละคนทำและส่งงานของตัวเอง ใน fork ของตัวเอง*

**Scenario (A, B or C) / Scenario ที่เลือก:** A. Owner alerts

Choose **one** scenario below. If the last digit of your student ID is odd, take A or B; if it is even, take B or C. Use your measurements from Tutorial 6 as evidence: cite run names from `results/runs.csv`. Answer in Thai or English.

เลือก scenario **หนึ่ง** ข้อด้านล่าง ถ้าเลขตัวสุดท้ายของรหัสนิสิตเป็นเลขคี่ ให้เลือก A หรือ B ถ้าเป็นเลขคู่ ให้เลือก B หรือ C ใช้ผลการวัดจาก tutorial ที่ทำไปเป็นข้อมูลอ้างอิง โดยอ้างชื่อ run จาก `results/runs.csv` จะตอบเป็นภาษาไทยหรือภาษาอังกฤษก็ได้

Facts that hold in every scenario / ข้อเท็จจริงที่ใช้กับทุก scenario:

- The large model serves at most **20 calls per second** (4 calls at once × about 200 ms each), and every call costs money.

  *large model รับได้สูงสุด **20 calls ต่อวินาที** (พร้อมกัน 4 calls × ประมาณ 200 ms ต่อ call) และทุก call มีค่าใช้จ่าย*
- The small model is fast and cheap enough to run anywhere, including inside the review service or on a kiosk, but it is much less accurate on its own.

  *small model เร็วและถูกพอที่จะรันที่ไหนก็ได้ รวมถึงภายใน review service หรือบน kiosk แต่ถ้าใช้ตัวเดียวจะแม่นน้อยกว่ามาก*

## Scenarios

**A. Owner alerts.** When a customer posts a negative review, the restaurant owner gets a push notification within **2 seconds**. At the lunch peak, CampusEats receives **40 reviews per second**. A missed alert is acceptable, but a wrong alert (for example, "your customer is unhappy" when they are happy) makes owners stop trusting alerts and, in the end, ignore them.

*เมื่อลูกค้าโพสต์รีวิวเชิงลบ เจ้าของร้านจะต้องได้รับ push notification ภายใน **2 วินาที** เวลาช่วงพีคตอนเที่ยง CampusEats ได้รับรีวิวจำนวน **40 รายการต่อวินาที** การพลาด alert ไปบ้างนั้นยอมรับได้ แต่ alert ที่ผิด (เช่น "ลูกค้าไม่พอใจ" ทั้งที่จริงๆ แล้วพอใจ) จะทำให้เจ้าของร้านไม่เชื่อถือและเลิกสนใจ alert ในที่สุด*

**B. Canteen kiosk.** A tablet in each canteen lets students leave a review on the spot. However, the canteen Wi-Fi is poor: the tablets lose their connection several times a day, for minutes at a time. The system must accept every review, even while a tablet is offline. Restaurant owners need the labels **by the end of the day**.

*มี tablet ในแต่ละโรงอาหารให้นิสิตรีวิวได้ทันที อย่างไรก็ดี การเชื่อมต่อ Wi-Fi ในโรงอาหารไม่ดี ทำให้ tablet หลุดการเชื่อมต่อวันละหลายครั้ง ครั้งละหลายนาที ระบบจะต้องรับรีวิวได้ทุกรายการแม้ขณะที่กำลัง offline อยู่ และเจ้าของร้านต้องการ label **ภายในสิ้นวัน***

**C. Weekly report.** Restaurant owners want a weekly sentiment report. Nothing in producing this report has to be real-time. **Accuracy matters most** and **cost must be minimal**.

*เจ้าของร้านต้องการรายงาน sentiment รายสัปดาห์ ไม่มีอะไรที่จะต้องทำแบบ real-time ในการออกรายงานนี้ ความแม่นยำ **(accuracy) สำคัญที่สุด** และ **ค่าใช้จ่ายต้องน้อยที่สุด***

## Topologies and patterns to choose from / Topology และ pattern ที่เลือกใช้ได้

| | Topology | Where the model runs / โมเดลรันที่ไหน |
|---|---|---|
| T1 | Model as a service, synchronous | Inference service, called for each review (today's baseline)<br>*inference service ถูกเรียกทุกรีวิว (baseline ของวันนี้)* |
| T2 | Model as a library | Inside the review service, or on the device<br>*ภายใน review service หรือบนอุปกรณ์* |
| T3 | Hybrid | Small model locally, large model as a remote service<br>*small model รันในเครื่อง large model เป็น remote service* |
| T4 | Asynchronous / batch | Reviews are queued (broker, W10) or stored, and scored later in bulk<br>*รีวิวถูกเข้าคิว (broker, W10) หรือเก็บไว้ แล้วติด label ทีหลังเป็นชุด* |

Combine one topology with any of the patterns from Tutorial 6: **fallback**, **cascade** and **cache**.

*เลือก topology หนึ่งแบบ แล้วผสมกับ pattern จาก Tutorial 6 ได้ตามต้องการ: **fallback**, **cascade** และ **cache***

## 1. Quality requirements

The three most important quality requirements for your scenario, each with a number where possible.

*quality requirements ที่สำคัญที่สุด 3 ข้อของ scenario ที่เลือก ระบุตัวเลขเมื่อทำได้*

| # | Quality | Requirement |
|---|---|---|
| 1 | Performance | When a customer posts a negative review, the owner gets the push notification within **2 s** |
| 2 | Capacity | The system handles **40 reviews/s** at the lunch peak, while the large model serves at most **20 calls/s** |
| 3 | Accuracy | At least **90%** of labels are correct; a wrong alert is worse than a missed one |

## 2. Design / การออกแบบ

Your topology, the patterns you add, and a container-level diagram (C4 level 2). The C4 diagram must be written in Mermaid.

*topology ที่เลือก pattern ที่เพิ่ม และ diagram ระดับ container (C4 level 2) เขียนด้วย Mermaid*

**Topology:** T3, hybrid: the small model runs as a library inside the review service, and the large model stays a remote service.

**Patterns:**
- **Cascade (threshold 0.7):** the review service asks the local small model first. Only reviews with confidence below 0.7 go to the inference service.
- **Cache:** the inference service checks the prediction cache before calling the large model.
- **Fallback:** the review service waits at most 1 s for the inference service. On a timeout or an error, the review is saved as `pending` and no alert is sent (a missed alert is acceptable; a wrong one is not).

```mermaid
C4Container
    title Scenario A (owner alerts): container diagram
    Person(customer, "Customer", "Writes restaurant reviews")
    Person(owner, "Restaurant owner", "Gets an alert for each negative review")
    System_Boundary(campuseats, "CampusEats") {
        Container(gateway, "API gateway", "Python, FastAPI", "Single entry point for clients")
        Container(review, "Review service", "Python, FastAPI, small model as a library", "Stores reviews. Cascade: small model first. Confidence 0.7 or more is final, otherwise asks the inference service. Fallback: 1 s timeout, then pending and no alert")
        Container(inference, "Inference service", "Python, FastAPI, scikit-learn", "Prediction cache (LRU) in front of the large model. At most 20 calls per second")
        Container(notify, "Notification service", "Python", "Sends a push alert when a review is labelled negative")
    }
    Rel(customer, gateway, "Posts reviews", "HTTP/JSON")
    Rel(gateway, review, "Forwards requests", "HTTP/JSON")
    Rel(review, inference, "POST /predict, only when unsure", "REST, 1 s timeout")
    Rel(review, notify, "Negative label", "Event")
    Rel(notify, owner, "Push alert within 2 s", "Push notification")
    UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")
```

## 3. Evidence / หลักฐาน

Which runs support your choice? Show the numbers and, for scenario A, the capacity arithmetic: how many large-model calls per second does your design need at the peak?

*run ไหนที่สนับสนุนการตัดสินใจของนิสิต ให้นิสิตแสดงตัวเลขประกอบ และสำหรับ scenario A ให้นิสิตแสดงการคำนวณ capacity ด้วย เพื่อแสดงว่าในช่วงพีคนั้น design ของนิสิตต้องใช้ large model กี่ calls ต่อวินาที*

| Run | What it shows / แสดงอะไร |
|---|---|
| `t0-large` | Accurate (93.5%), but 100% large calls → 40 × 1.00 = **40 calls/s** at the peak, twice the 20 calls/s limit. Large-only fails capacity. |
| `t0-small` | 0 calls/s, but only 72.5% accuracy. Small-only fails the 90% requirement and would send many wrong alerts. |
| `t2-0.6` | 38% large calls → **15.2 calls/s** ✅ and 91.5% ✅, but only 1.5 points above the 90% limit, too little margin when wrong alerts are costly. |
| `t2-0.7` | 93.0% ✅, but 53% large calls → **21.2 calls/s**, just over the limit ❌. The cascade alone is not enough at 40 reviews/s. |
| `t3-cache` | Cascade 0.7 + cache: 38% large calls → **15.2 calls/s** ✅ (25% headroom below 20) and 93.0% accuracy ✅. Only this option passes both, with a margin. p95 580 ms, well inside the 2 s alert deadline. |
| `t1-after` | With fallback, a hanging large model gives 0 errors and 24 pending instead of 20 errors, so a hang causes missed alerts, not a stalled system. |

**Capacity at the peak (my design):** 40 reviews/s × 38% large calls = **15.2 large-model calls/s ≤ 20 calls/s** ✅

## 4. ADR

Use the ADR format from W5.

*ใช้รูปแบบ ADR จาก W5*

**Title:** Hybrid cascade with cache and fallback for owner alerts

**Status:** Proposed

**Context:** Restaurant owners must get a push alert within 2 s of a negative review. At the lunch peak 40 reviews/s arrive, but the large model serves at most 20 calls/s, so sending every review to it is impossible (`t0-large`: 40 calls/s). The small model alone is fast but only 72.5% accurate (`t0-small`), below the 90% requirement. A wrong alert makes owners stop trusting alerts, while a missed alert is acceptable.

**Decision:** Use T3 (hybrid). The small model runs as a library inside the review service. Reviews with confidence ≥ 0.7 are labelled locally; the rest go to the remote large model through the inference service, which checks an LRU prediction cache first. The review service waits at most 1 s for the inference service; on a timeout or an error, the review is saved as `pending` and no alert is sent.

**Consequences:**
- Positive: at the peak the large model needs 40 × 38% = 15.2 calls/s, within the 20 calls/s limit (`t3-cache`). Accuracy stays at 93.0%, 3 points above the limit, and p95 is 580 ms, well inside the 2 s deadline. A hanging large model causes missed alerts instead of stalling the whole system (`t1-after`).
- Negative: when the fallback kicks in, the review is saved as `pending` and the owner gets **no alert** for that negative review. Its label only arrives later from a batch job, so the owner can miss an unhappy customer.

**Alternatives considered:** at least one topology you rejected, and why.

*อย่างน้อยหนึ่ง topology ที่ไม่เลือก และเหตุผล*

> **T4 (asynchronous / batch)** was rejected: reviews are queued and scored later in bulk, so it cannot send an alert within 2 s.
