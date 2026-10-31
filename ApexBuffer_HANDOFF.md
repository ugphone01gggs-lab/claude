# ApexBuffer: คำอธิบายปัญหาสำหรับการแก้ไขต่อ

เอกสารนี้สรุปสิ่งที่ผู้ใช้พยายามทำ สิ่งที่ทำไปแล้ว อาการที่เกิดขึ้น และคำถามที่ต้องการคำตอบ เขียนให้อ่านได้โดยไม่ต้องรู้ประวัติการสนทนาก่อนหน้า

---

## 1. เป้าหมายของผู้ใช้

ผู้ใช้เป็นนักพัฒนา mod ของเกม **Geometry Dash** บนมือถือ Android โดยใช้ **Geode** (mod loader)
ต้องการทำ mod ชื่อ **ApexBuffer** ซึ่งเป็น input buffer (ระบบจัดการการกดปุ่มให้แม่นขึ้น) ที่:

1. ทำงานได้บน Android (และ Windows ตามที่ตั้งไว้ใน build)
2. มี **หน้าตั้งค่า (settings popup)** ที่เปิดได้จากในเกม
3. ผู้ใช้ **ไม่ได้เป็นโปรแกรมเมอร์โดยตรง** จึงต้องการให้แก้ไขโค้ดใน repo บน GitHub โดยมี build อัตโนมัติผ่าน GitHub Actions

ผู้ใช้เคยพยายามใช้ mod อีกตัวชื่อ **Predictive Buffer / BASU** แต่หน้าเมนูไม่ขึ้น จึงตัดสินใจทำ ApexBuffer ขึ้นใหม่เป็นตัวหลัก

---

## 2. สภาพแวดล้อมและการตั้งค่า (ตรวจแล้ว)

| รายการ | ค่า |
|---|---|
| ตัวเกม | Geometry Dash **2.2081** (Android) |
| Geode SDK | **5.11.2** (ใช้ `sdk: stable` ใน workflow) |
| ภาษา C++ | **C++23** (`CMAKE_CXX_STANDARD 23`) |
| mod id | `apex.buffer` |
| version | `v1.5.0` |
| target ที่ build | Windows, Android32, Android64 (รวมไฟล์ด้วย `combine: true`) |

โครงสร้างโปรเจกต์:
```
ApexBuffer/
  CMakeLists.txt          (C++23, setup_geode_mod)
  mod.json                (geode 5.11.2, gd 2.2081, settings ทั้งหมด)
  src/main.cpp            (โค้ดหลักทั้งหมด ~400 บรรทัด)
  resources/              (ว่างเปล่า ไม่มีรูปใดๆ ใช้ไม่ได้)
  .github/workflows/build.yml
```

**build ผ่านแล้ว** ตามผลล่าสุดของ GitHub Actions (ก่อนเพิ่มปุ่มเมนูหลัก) ดังนั้นปัญหาตอนนี้ไม่ใช่ compile error แต่เป็นปัญหาเรื่องการแสดงผลและการทำงานของ UI

---

## 3. สิ่งที่ทำไปแล้ว (ตามลำดับ)

1. **ปุ่มในหน้า Pause (PauseLayer)**: ทดลองหลายแบบ
   - ใช้ `CCMenu` แยกที่พิกัดคงที่ มุมล่างขวา (`win.width - 50, 30`) ของ BASU → **ไม่ขึ้น**
   - พยายามใส่เข้า `m_buttonMenu` ของ PauseLayer → **ไม่แน่ใจว่ามีสมาชิกชื่อนี้จริง** (เอกสาร Geode ที่พบแสดงสมาชิกนี้ใน SlideInLayer/Popup ไม่ใช่ PauseLayer) จึงเอาออก
   - ปัจจุบันใช้ CCMenu แยกที่ `{win.width - 55, win.height * 0.2}` → **ผู้ใช้ยืนยันว่าไม่ขึ้น**
2. **ปุ่มในหน้าเมนูหลัก (MenuLayer)** (v1.5.0): สร้าง CCMenu แยกที่ `{0,0}` แล้วใส่ปุ่มที่ `{win.width - 60, 60}` ด้วย z-order 100 → **ผู้ใช้ยืนยันว่าไม่ขึ้น** (ดูหัวข้อ 4)
3. **คีย์ลัด X** ผ่าน `listenForKeybindSettingPresses("open-menu-key", ...)` และ setting ใน mod.json ชนิด `keybind` ค่าเริ่มต้น `["X"]` → **กดแล้วไม่มีอะไรเกิดขึ้น**
4. แก้ไขการ override `pushButton`/`releaseButton` ให้คืนค่า `bool` (ตรงกับลายเซ็นของเกม) ซึ่งทำให้ build ผ่าน
5. แก้ `gd` ใน mod.json จาก 2.2074 เป็น 2.2081 เพราะ SDK ไม่ตรงกับเวอร์ชันเกม (เป็นสาเหตุที่ build พังก่อนหน้า)

---

## 4. อาการปัจจุบัน (สิ่งที่ผู้ใช้เห็นจริง)

- **หน้าเมนูหลักของเกม: ไม่มีปุ่ม Apex** ทั้งที่ build v1.5.0 แล้ว
- **หน้า Pause ของด่าน: ไม่มีปุ่ม Apex**
- **กดปุ่ม X บนคีย์บอร์ด: ไม่มีอะไรเกิดขึ้น**
- ผู้ใช้เคยได้เมนู "Mega Hack" ซึ่งไม่เกี่ยวข้อง

### ข้อมูลจาก log ของ Geode (รอบล่าสุด)
- `Loading mod apex.buffer v1.4.1` → **โหลดสำเร็จ ไม่มี error ของตัวเอง**
  - หมายเหตุ: log ยังแสดง `v1.4.1` ซึ่งหมายความว่าเกมโหลด **build เก่า** ไม่ใช่ v1.5.0 ที่เพิ่มปุ่มเมนู (version ในไฟล์ที่ติดตั้งจริงต้องตรวจก่อน)
- `Unable to load setting 'open-menu-key' ... not found in savedata` → ปกติในการเปิดครั้งแรก ไม่ใช่ error จริง
- ไม่มีบรรทัด `ApexBuffer v1.5: main menu button added` (ซึ่งจะขึ้นถ้า MenuLayer hook ทำงานและเพิ่มปุ่มแล้ว)
- ไม่มีบรรทัดจากการเข้าด่านหรือการ Pause เลย (ยังไม่ได้ทดสอบจริงในด่าน)
- มี mod อื่นที่ทำงานกับ input และเสียงโหลดพร้อมกัน: `basu.input-buffer v2.87.0`, `syzzi.click_between_frames`, `beat.click-sound`, `geode.custom-keybinds` ซึ่งอาจชนกันหรือแย่ง key X ได้

---

## 5. โค้ดปัจจุบันของ MenuLayer hook (ที่ไม่ขึ้น)

```cpp
class $modify(ApexMenuLayer, MenuLayer) {
    bool init() {
        if (!MenuLayer::init()) return false;

        auto* spr = ButtonSprite::create("Apex", "bigFont.fnt", "GJ_button_02.png", 0.6f);
        auto* btn = CCMenuItemSpriteExtra::create(spr, this, menu_selector(ApexMenuLayer::onApexBtn));
        btn->setID("apex-menu-btn"_spr);

        auto* menu = CCMenu::create();
        menu->setPosition({0.f, 0.f});
        menu->setID("apex-menu"_spr);
        this->addChild(menu, 100);

        auto win = CCDirector::sharedDirector()->getWinSize();
        btn->setPosition({win.width - 60.f, 60.f});
        menu->addChild(btn);

        log::info("ApexBuffer v1.5: main menu button added");
        return true;
    }

    void onApexBtn(CCObject*) {
        geode::openSettingsPopup(Mod::get());
    }
};
```

ข้อสังเกต:
- ไม่มี `$execute` ใดที่เรียก `log::info` เพื่อยืนยันว่า mod ถูก initialize
- ไม่มีการตรวจว่า `MenuLayer::init()` ถูกเรียกหรือไม่ และไม่ได้ตรวจว่า hook ถูกใช้จริง

---

## 6. ปุ่มในหน้า Pause (โค้ดปัจจุบัน)

```cpp
class $modify(ApexPauseLayer, PauseLayer) {
    void customSetup() {
        PauseLayer::customSetup();
        auto* spr = ButtonSprite::create("Apex", "bigFont.fnt", "GJ_button_02.png", 0.5f);
        auto* btn = CCMenuItemSpriteExtra::create(spr, this, menu_selector(ApexPauseLayer::onApexBtn));
        auto* menu = CCMenu::create();
        menu->setPosition({0.f, 0.f});
        this->addChild(menu, 100);
        auto win = CCDirector::sharedDirector()->getWinSize();
        btn->setPosition({win.width - 55.f, win.height * 0.2f});
        menu->addChild(btn);
    }
    void onApexBtn(CCObject*) { geode::openSettingsPopup(Mod::get()); }
};
```

---

## 7. คำถามที่ต้องการคำตอบ (ตามลำดับความสำคัญ)

1. **ทำไม hook ของ MenuLayer ไม่ทำงาน?** ตรวจว่าการ override `bool init()` ใน `$modify(ApexMenuLayer, MenuLayer)` เป็นลายเซ็นที่ถูกต้องใน Geode 5.11.2 หรือไม่ และ `init` ถูกเรียกจริงหรือไม่ (ควรเพิ่ม log ที่บรรทัดแรกของ init)
2. **ทำไม build ที่ติดตั้งยังแสดง v1.4.1?** ตรวจว่า workflow รันบน commit ล่าสุด, artifact ที่ดาวน์โหลดเป็นของรอบล่าสุด, และ mod.json ที่ถูก commit มี version `v1.5.0` จริง
3. **ควรวางปุ่มอย่างไรให้เห็นจริงบนมือถือ?** ใช้ CCMenu แยกที่พิกัดคงที่ยังไม่ได้ผลทั้งใน PauseLayer และ MenuLayer ควรหาวิธีที่ Geode แนะนำ เช่น การเพิ่มปุ่มเข้าเมนูที่มีอยู่แล้วของเกม (ต้องยืนยันชื่อสมาชิกหรือ node id ที่มีอยู่จริง) และใช้ `updateLayout()` ถ้าเหมาะสม
4. **คีย์ลัด X ทำงานบน Android หรือไม่?** GD บนมือถืออาจไม่ส่งเหตุการณ์คีย์บอร์ดให้ mod ถ้าเป็นเช่นนั้น ควรเน้นปุ่มบนหน้าจอแทน
5. **mod อื่นที่ซ้อนกันทำให้ปุ่มหายไปหรือไม่?** โดยเฉพาะ `absolllute.megahack` (ตอนนี้ปิดแล้วในการทดสอบล่าสุด) และ mod ที่แก้หน้าเมนูหรือ pause เหมือนกัน

---

## 8. ข้อจำกัดและหลักการ

- **ไม่มี compiler หรือ Geode SDK ในเครื่องผู้ช่วยนี้** ต้องให้ GitHub Actions compile เท่านั้น ควรเขียนโค้ดให้ปลอดภัยที่สุด (ใช้เฉพาะ API ที่มั่นใจ)
- ก่อนอ้างชื่อสมาชิกของคลาสเกม (เช่น `m_buttonMenu`) ควรยืนยันจากเอกสาร Geode หรือ bindings ของเวอร์ชันนั้นก่อน เพราะเคยเกิดปัญหาที่ชื่อสมาชิกไม่มีจริง
- ใน `$modify` ห้ามประกาศตัวแปรสมาชิกโดยตรง ต้องใช้ `struct Fields` และ `m_fields->...` (โค้ดปัจจุบันทำถูกแล้ว)
- ผู้ใช้เป็นคนทดสอบบนมือถือจริงและส่ง log/ภาพหน้าจอกลับมา ควรเพิ่ม log ที่ชัดเจนในแต่ละจุดที่ควรทำงาน เพื่อให้วินิจฉัยได้จาก log โดยไม่ต้องเดา
- ผู้ใช้ไม่ใช่โปรแกรมเมอร์ จึงควรให้คำอธิบายแบบเข้าใจง่าย และให้ไฟล์ที่พร้อมวางแทนที่ได้ทันที

---

## 9. สิ่งที่ผู้ใช้ต้องการจากการแก้รอบนี้

- ปุ่ม **Apex** ต้อง**เห็นได้จริง**บนหน้าเมนูหลัก หรือหน้า Pause (อย่างน้อยหนึ่งจุด)
- กดปุ่มแล้ว**หน้าตั้งค่า ApexBuffer เปิดขึ้น**
- มี log ที่ยืนยันทุกขั้นตอนสำคัญ
- ส่งมอบเป็น **ไฟล์ที่วางแทนที่ใน repo ได้เลย** พร้อมคำอธิบายสั้นๆ ว่าเปลี่ยนอะไร
