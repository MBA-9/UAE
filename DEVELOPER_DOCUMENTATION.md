# توثيق نظام أوقات الصلاة - UAE Prayer Times System

## نظرة عامة / Overview

تم تحويل ملفات Word التي تحتوي على أوقات الصلاة إلى هيكلة JSON منظمة ورفعها على GitHub لسهولة الوصول والاستخدام في تطبيقات iOS (Xcode).

Word documents containing prayer times have been converted to organized JSON structure and uploaded to GitHub for easy access and use in iOS (Xcode) applications.

---

## هيكلة النظام / System Structure

```
uae/
├── metadata.json                    # قائمة الإمارات والمناطق
│                                    # List of Emirates and Regions
└── times/
    └── 2026/
        ├── abu_dhabi/
        │   ├── abu_dhabi_city.json
        │   └── al_ain.json
        ├── dubai/
        │   └── dubai_city.json
        └── sharjah/
            └── sharjah_city.json
```

---

## ملف Metadata / Metadata File

### الرابط Raw:
```
https://raw.githubusercontent.com/MBA-9/UAE/main/uae/metadata.json
```

### البنية / Structure:
```json
{
  "emirates": [
    {
      "id": "abu_dhabi",
      "name_ar": "أبو ظبي",
      "name_en": "Abu Dhabi",
      "regions": [
        {
          "id": "abu_dhabi_city",
          "name_ar": "أبو ظبي",
          "name_en": "Abu Dhabi City"
        },
        {
          "id": "al_ain",
          "name_ar": "العين",
          "name_en": "Al Ain"
        }
      ]
    }
  ]
}
```

---

## ملفات أوقات الصلاة / Prayer Times Files

### أبو ظبي - Abu Dhabi

#### أبو ظبي المدينة / Abu Dhabi City:
```
https://raw.githubusercontent.com/MBA-9/UAE/main/uae/times/2026/abu_dhabi/abu_dhabi_city.json
```

#### العين / Al Ain:
```
https://raw.githubusercontent.com/MBA-9/UAE/main/uae/times/2026/abu_dhabi/al_ain.json
```

### دبي - Dubai

#### دبي المدينة / Dubai City:
```
https://raw.githubusercontent.com/MBA-9/UAE/main/uae/times/2026/dubai/dubai_city.json
```

### الشارقة - Sharjah

#### الشارقة المدينة / Sharjah City:
```
https://raw.githubusercontent.com/MBA-9/UAE/main/uae/times/2026/sharjah/sharjah_city.json
```

---

## بنية ملف أوقات الصلاة / Prayer Times File Structure

كل ملف يحتوي على مصفوفة من الكائنات، كل كائن يمثل يوم واحد:

Each file contains an array of objects, each object represents one day:

```json
[
  {
    "date": "2026-01-01",
    "source": "GAIAEZ",
    "timezone": "Asia/Dubai",
    "fajr": "05:42",
    "sunrise": "07:00",
    "dhuhr": "12:25",
    "asr": "15:22",
    "maghrib": "17:44",
    "isha": "19:03"
  }
]
```

### الحقول / Fields:

- **date**: التاريخ بصيغة ISO (YYYY-MM-DD) / Date in ISO format (YYYY-MM-DD)
- **source**: مصدر البيانات (GAIAEZ) / Data source (GAIAEZ)
- **timezone**: المنطقة الزمنية (Asia/Dubai) / Timezone (Asia/Dubai)
- **fajr**: وقت الفجر (24 ساعة) / Fajr time (24-hour format)
- **sunrise**: وقت الشروق / Sunrise time
- **dhuhr**: وقت الظهر / Dhuhr time
- **asr**: وقت العصر / Asr time
- **maghrib**: وقت المغرب / Maghrib time
- **isha**: وقت العشاء / Isha time

---

## كيفية الاستخدام في Xcode / How to Use in Xcode

### 1. جلب البيانات من GitHub / Fetch Data from GitHub

#### Swift Example:

```swift
import Foundation

struct PrayerTime: Codable {
    let date: String
    let source: String
    let timezone: String
    let fajr: String
    let sunrise: String
    let dhuhr: String
    let asr: String
    let maghrib: String
    let isha: String
}

struct EmiratesMetadata: Codable {
    let emirates: [Emirate]
}

struct Emirate: Codable {
    let id: String
    let nameAr: String
    let nameEn: String
    let regions: [Region]
    
    enum CodingKeys: String, CodingKey {
        case id
        case nameAr = "name_ar"
        case nameEn = "name_en"
        case regions
    }
}

struct Region: Codable {
    let id: String
    let nameAr: String
    let nameEn: String
    
    enum CodingKeys: String, CodingKey {
        case id
        case nameAr = "name_ar"
        case nameEn = "name_en"
    }
}

class PrayerTimesService {
    static let baseURL = "https://raw.githubusercontent.com/MBA-9/UAE/main"
    
    // جلب قائمة الإمارات / Fetch Emirates List
    static func fetchMetadata(completion: @escaping (Result<EmiratesMetadata, Error>) -> Void) {
        let url = URL(string: "\(baseURL)/uae/metadata.json")!
        
        URLSession.shared.dataTask(with: url) { data, response, error in
            if let error = error {
                completion(.failure(error))
                return
            }
            
            guard let data = data else {
                completion(.failure(NSError(domain: "NoData", code: -1)))
                return
            }
            
            do {
                let metadata = try JSONDecoder().decode(EmiratesMetadata.self, from: data)
                completion(.success(metadata))
            } catch {
                completion(.failure(error))
            }
        }.resume()
    }
    
    // جلب أوقات الصلاة لمنطقة معينة / Fetch Prayer Times for Specific Region
    static func fetchPrayerTimes(
        emirate: String,
        region: String,
        year: Int = 2026,
        completion: @escaping (Result<[PrayerTime], Error>) -> Void
    ) {
        let url = URL(string: "\(baseURL)/uae/times/\(year)/\(emirate)/\(region).json")!
        
        URLSession.shared.dataTask(with: url) { data, response, error in
            if let error = error {
                completion(.failure(error))
                return
            }
            
            guard let data = data else {
                completion(.failure(NSError(domain: "NoData", code: -1)))
                return
            }
            
            do {
                let prayerTimes = try JSONDecoder().decode([PrayerTime].self, from: data)
                completion(.success(prayerTimes))
            } catch {
                completion(.failure(error))
            }
        }.resume()
    }
    
    // البحث عن أوقات الصلاة لتاريخ معين / Find Prayer Times for Specific Date
    static func findPrayerTime(
        for date: Date,
        in prayerTimes: [PrayerTime]
    ) -> PrayerTime? {
        let formatter = DateFormatter()
        formatter.dateFormat = "yyyy-MM-dd"
        let dateString = formatter.string(from: date)
        
        return prayerTimes.first { $0.date == dateString }
    }
}
```

### 2. مثال على الاستخدام / Usage Example:

```swift
// جلب قائمة الإمارات / Fetch Emirates
PrayerTimesService.fetchMetadata { result in
    switch result {
    case .success(let metadata):
        print("Emirates: \(metadata.emirates.count)")
        for emirate in metadata.emirates {
            print("\(emirate.nameEn) - \(emirate.nameAr)")
        }
    case .failure(let error):
        print("Error: \(error)")
    }
}

// جلب أوقات الصلاة لدبي / Fetch Prayer Times for Dubai
PrayerTimesService.fetchPrayerTimes(
    emirate: "dubai",
    region: "dubai_city"
) { result in
    switch result {
    case .success(let times):
        print("Found \(times.count) days of prayer times")
        
        // البحث عن أوقات اليوم / Find today's prayer times
        if let today = PrayerTimesService.findPrayerTime(
            for: Date(),
            in: times
        ) {
            print("Fajr: \(today.fajr)")
            print("Dhuhr: \(today.dhuhr)")
            print("Maghrib: \(today.maghrib)")
        }
    case .failure(let error):
        print("Error: \(error)")
    }
}
```

---

## ملخص العملية / Process Summary

### الخطوات المتبعة / Steps Taken:

1. **استخراج البيانات / Data Extraction**:
   - استخدام مكتبة `python-docx` لقراءة ملفات Word
   - تحليل النص العربي وتحديد التواريخ والأوقات
   - تحويل التواريخ من الصيغة العربية (01 يناير 2026) إلى ISO (2026-01-01)
   - تحويل الأوقات من 12 ساعة (05:42 ص) إلى 24 ساعة (05:42)

2. **تنظيم البيانات / Data Organization**:
   - إنشاء هيكلة المجلدات حسب المواصفات
   - إنشاء ملف `metadata.json` يحتوي على قائمة الإمارات والمناطق
   - إنشاء ملفات JSON منفصلة لكل منطقة تحتوي على أوقات الصلاة

3. **الرفع على GitHub / GitHub Upload**:
   - تهيئة مستودع Git محلي
   - إضافة الملفات والالتزام بها
   - رفع الملفات إلى المستودع على GitHub

4. **التوثيق / Documentation**:
   - إنشاء ملف توثيق شامل
   - توفير روابط Raw لجميع الملفات
   - إضافة أمثلة كود Swift للاستخدام في Xcode

---

## الإحصائيات / Statistics

- **عدد الإمارات / Number of Emirates**: 3
  - أبو ظبي / Abu Dhabi
  - دبي / Dubai
  - الشارقة / Sharjah

- **عدد المناطق / Number of Regions**: 4
  - أبو ظبي المدينة / Abu Dhabi City: 298 يوم
  - العين / Al Ain: 298 يوم
  - دبي المدينة / Dubai City: 180 يوم
  - الشارقة المدينة / Sharjah City: 62 يوم

- **السنة / Year**: 2026

---

## ملاحظات مهمة / Important Notes

1. **التوقيت / Timezone**: جميع الأوقات تستخدم توقيت `Asia/Dubai`
2. **المصدر / Source**: جميع البيانات من مصدر `GAIAEZ`
3. **الصيغة / Format**: جميع الأوقات بصيغة 24 ساعة (HH:MM)
4. **التواريخ / Dates**: جميع التواريخ بصيغة ISO (YYYY-MM-DD)

---

## روابط سريعة / Quick Links

### Metadata:
- https://raw.githubusercontent.com/MBA-9/UAE/main/uae/metadata.json

### Prayer Times 2026:
- **Abu Dhabi City**: https://raw.githubusercontent.com/MBA-9/UAE/main/uae/times/2026/abu_dhabi/abu_dhabi_city.json
- **Al Ain**: https://raw.githubusercontent.com/MBA-9/UAE/main/uae/times/2026/abu_dhabi/al_ain.json
- **Dubai City**: https://raw.githubusercontent.com/MBA-9/UAE/main/uae/times/2026/dubai/dubai_city.json
- **Sharjah City**: https://raw.githubusercontent.com/MBA-9/UAE/main/uae/times/2026/sharjah/sharjah_city.json

---

## الدعم / Support

لأي استفسارات أو مشاكل، يرجى التواصل مع فريق التطوير.

For any inquiries or issues, please contact the development team.

---

**آخر تحديث / Last Updated**: 2026-01-01
