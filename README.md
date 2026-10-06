# TR

# SkyscannerREF ✈️

Excel üzerinden tanımlanan uçuş aramalarını Skyscanner üzerinde gerçekleştiren, sonuçları Excel'e aktaran **UiPath REFramework** tabanlı RPA projesi.

Her arama satırı Orchestrator kuyruğunda bir işlem olarak ele alınır. Robot kalkış, varış, tarih ve yolcu bilgilerini kullanarak **tek yön** uçuş araması yapar; her sefer için en fazla **10 sonuç** toplar.

## Özellikler

- Excel'den toplu uçuş arama verisi okuma.
- Orchestrator kuyruğuna toplu kayıt yükleme ve işlem durumu takibi.
- Chrome üzerinden Skyscanner'ın Türkçe arayüzünde arama.
- Yetişkin ve çocuk yolcu sayılarını ayrıştırma.
- Arama tarihinin bugün ile bir yıl sonrası arasında olduğunu kontrol etme.
- Kalkış yeri, varış yeri, firma ismi ve fiyat bilgilerini çıkarma.
- Sonuçları tarihli bir Excel dosyasında sefer adına göre ayrı sayfalara yazma.
- 10'dan az sonuç veya sonuç bulunamaması durumunda e-posta bildirimi.
- REFramework üzerinden günlük kaydı, hata yönetimi ve sistem hatalarında ekran görüntüsü alma.

## İşleyiş

```text
Config.xlsx yüklenir
        ↓
Kuyruk temizliği → Girdi Excel'i okunur → Kayıtlar kuyruğa yüklenir
        ↓
Skyscanner açılır → Kuyruktan işlem alınır
        ↓
Tarih ve yolcu bilgileri hazırlanır
        ↓
Tek yön araması yapılır → En fazla 10 uçuş sonucu çıkarılır
        ↓
Sonuçlar Excel'e yazılır → Kuyruk işlem durumu güncellenir
        ↓
Sonraki işlem / sürecin sonlandırılması
```

## Gereksinimler

- Windows ve Windows uyumlu projeleri destekleyen UiPath Studio / Robot.
- Projede kayıtlı Studio sürümü: `26.0.200.0`. Bağımlılık sürümlerini destekleyen bir Studio ortamı kullanılmalıdır.
- Microsoft Excel masaüstü uygulaması.
- Google Chrome ve etkin UiPath Chrome uzantısı.
- Orchestrator bağlantısı; kuyruk kayıtlarını okuma, ekleme, silme ve işlem durumlarını güncelleme yetkileri.
- E-posta bildirimleri için kullanıcının kendi Gmail / Google Workspace bağlantısı.
- Skyscanner, Orchestrator ve e-posta hizmetlerine internet erişimi.

Proje türü **Process**, hedef platform **Windows**, ifade dili **Visual Basic** ve ana giriş dosyası `Main.xaml`'dır. Tarayıcı otomasyonu için etkileşimli bir Windows oturumu gerekir.

### Paket bağımlılıkları

| Paket | Sürüm |
|---|---|
| UiPath.Excel.Activities | 3.6.1 |
| UiPath.GSuite.Activities | 3.11.10 |
| UiPath.Mail.Activities | 2.11.10 |
| UiPath.System.Activities | 26.6.3 |
| UiPath.Testing.Activities | 25.10.2 |
| UiPath.UIAutomation.Activities | 26.10.3 |

Bağımlılıklar `project.json` içinde tanımlıdır; projeyi açarken Studio üzerinden geri yüklenir.

## Kurulum

1. Depoyu klonlayın veya ZIP olarak indirip yerel bir klasöre çıkarın.
2. UiPath Studio'da `project.json` dosyasını açın ve paketlerin geri yüklenmesini bekleyin.
3. Chrome için UiPath uzantısını etkinleştirin.
4. Orchestrator'da bu projeye özel bir test kuyruğu oluşturun ve robotun ilgili klasöre erişimini sağlayın.
5. `Data/Config.xlsx` içindeki ayarları kendi ortamınıza göre düzenleyin.
6. `Init/Queue_Cleaning.xaml` içindeki **Get Queue Items → FolderPath** değerini kendi Orchestrator klasörünüzle değiştirin. Bu aktivitede geliştiriciye ait kişisel çalışma alanı sabit olarak kayıtlıdır; yalnızca Config değerini değiştirmek yeterli değildir.
7. `Process/ExtractPageData.xaml` içindeki e-posta aktivitelerini kendi bağlantınız ve alıcı adresinizle yapılandırın. Mevcut bağlantı kimliği ve Gmail hesap bilgileri başka ortamlara otomatik taşınmaz.
8. Girdi dosyasını hazırlayın ve `Main.xaml`'ı çalıştırın.

> **Çalıştırmadan önce:** Başlatma aşaması, `Queue_Cleaning.xaml` ile seçili kuyrukta `New` durumunda bulunan ve sorgudan dönen kayıtları siler. Paylaşılan veya üretim kuyruğu yerine bu projeye ayrılmış bir test kuyruğu kullanın. `Main.xaml` ayrıca tüm kapsam için `chrome.exe` sürecini sonlandırmaya yönelik bir aktivite içerir; çalıştırmadan önce açık Chrome oturumlarınızı kaydedin ve bu aktiviteyi ortamınıza göre kontrol edin.

## Yapılandırma

`Data/Config.xlsx` dosyasında `Settings`, `Constants` ve `Assets` sayfaları bulunur.

| Ayar | Varsayılan değer / kullanım |
|---|---|
| `OrchestratorQueueName` | `REF_Queue_Skyscanner` |
| `OrchestratorQueueFolder` | İlgili Orchestrator klasörü; mevcut dosyada boş |
| `SkyscannerInputExcelPath` | `Data\Input\SkyscannerInput.xlsx` |
| `SkyscannerWebsiteURL` | `https://www.skyscanner.com.tr/ucak-bileti` |
| `SkyscannerOutputExcelPath` | `Data\Output\SkyscannerOutput[date].xlsx` |
| `ExScreenshotsFolderPath` | `Exceptions_Screenshots` |
| `MaxRetryNumber` | `0`; kuyruk işlemlerinin yeniden deneme ayarları Orchestrator üzerinden kontrol edilmelidir |
| `MaxConsecutiveSystemExceptions` | `0`; ardışık sistem hatası sınırı devre dışı |

`Main.xaml` içinde `in_OrchestratorQueueName` ve `in_OrchestratorQueueFolder` argümanları da bulunur. Ancak başlangıçtaki kuyruk temizleme ve yükleme adımları, bu argümanlar Config'e uygulanmadan önce çalışır. İlk kurulumda kuyruk bilgilerini doğrudan `Config.xlsx` üzerinden belirleyin ve sabit `FolderPath` değerini aynı klasörle eşleştirin.

## Girdi formatı

Varsayılan dosya: `Data/Input/SkyscannerInput.xlsx`. Robot çalışma kitabının **ilk sayfasını** okur ve hücrelerin görüntülenen değerlerini kullanır.

| Sütun | Açıklama | Örnek |
|---|---|---|
| `SEFER` | Sonuç Excel'indeki sayfa adı | `1. Sefer` |
| `KALKIŞ` | Kalkış şehri / havaalanı | `Ankara` |
| `VARIŞ` | Varış şehri / havaalanı | `İzmir` |
| `TARİH` | `d.MM.yyyy` biçiminde uçuş tarihi | `7.11.2026` |
| `KİŞİ SAYISI` | Yetişkin ve isteğe bağlı çocuk sayısı | `2 Yetişkin-1 Çocuk` |

- Sütun adlarını Türkçe karakterleriyle birlikte aynen koruyun.
- Excel tarih hücrelerinin görüntü biçimini `d.MM.yyyy` olarak ayarlayın. Örnek tarihi çalıştırma gününe göre güncelleyin.
- Tarih, çalıştırma gününden önce veya bir yıldan sonra olamaz.
- Yalnızca yetişkin için `2 Yetişkin`; çocuk içeren arama için `2 Yetişkin-1 Çocuk` biçimini kullanın.
- Mevcut akış tüm çocukların yaşını **7** olarak seçer; yaşlar girdi dosyasından alınmaz.
- `SEFER` değerlerini benzersiz ve Excel sayfa adı kurallarına uygun tutun.
- Depodaki örnek tarihler her çalıştırma öncesinde kontrol edilmelidir.

## Çıktı

Varsayılan çıktı adı `SkyscannerOutput[date].xlsx` olup `[date]` çalışma gününün `ddMMyyyy` değeriyle değiştirilir. Örneğin:

```text
Data/Output/SkyscannerOutput06102026.xlsx
```

Her seferin sonuçları, `SEFER` alanıyla aynı adlı sayfaya yazılır:

| Kalkış Yeri | Varış Yeri | Firma İsmi | Fiyat |
|---|---|---|---|
| ESB | ADB | Örnek Havayolu | 3.500 TL |

Tablodaki satır yalnızca çıktı biçimini gösteren temsili bir örnektir. Firma adı boş gelen sonuçlar `1+ Firma(Aktarmalı)` olarak doldurulur. Fiyatlar web sayfasında görünen metin biçiminde aktarılır.

Aynı gün ve aynı `SEFER` adıyla yeniden çalıştırmada aynı dosya ve sayfa hedeflenir; geçmiş sonuçları korumak için çıktı dosyasını önceden yedekleyin.

## Proje yapısı

```text
SkyscannerREF/
├── Main.xaml                         # REFramework durum makinesi
├── project.json                      # Proje bilgileri ve bağımlılıklar
├── Framework/                        # İşlem alma, işleme ve durum yönetimi
├── Init/
│   ├── Read_Excel.xaml                # Girdi verisini okuma
│   ├── Queue_Cleaning.xaml            # Bekleyen kuyruk kayıtlarını temizleme
│   ├── Queue_Loading.xaml             # Excel satırlarını kuyruğa yükleme
│   └── Open_Website.xaml              # Skyscanner'ı açma
├── Process/
│   ├── DataValidationAndManipulation.xaml
│   ├── NavigateToFilterPage.xaml
│   ├── FilterPage.xaml
│   ├── ExtractPageData.xaml
│   └── Write_Excel.xaml
├── Data/
│   ├── Config.xlsx
│   ├── Input/SkyscannerInput.xlsx
│   ├── Output/
│   └── Temp/
├── Tests/                            # REFramework testleri ve şablonları
├── Documentation/                    # Framework dokümantasyonu
└── Exceptions_Screenshots/            # Hata ekran görüntüleri
```

## Hata yönetimi ve mevcut sınırlamalar

- Girdi dosyasının bulunamaması, geçersiz tarih aralığı ve uçuş bulunamaması için hata dalları tanımlıdır.
- REFramework, işlem sonucunu kuyrukta başarı / iş kuralı hatası / sistem hatası olarak yönetir ve sistem hatalarında ekran görüntüsü almaya çalışır.
- Bazı alt akışlardaki genel `Exception` yakalama blokları hataları `SystemException` olarak yeniden fırlatır. Bu nedenle içte tanımlanan bir iş kuralı hatasının kuyrukta her zaman Business Exception olarak işaretleneceği varsayılmamalıdır.
- Arama Türkçe arayüz ve tek yön seçimi için hazırlanmıştır. Site tasarımındaki değişiklikler, çerez pencereleri veya CAPTCHA akışın güncellenmesini gerektirebilir.
- Çıkarma işlemi en fazla 10 satırla sınırlıdır; tüm sonuçların taranması veya en ucuz 10 uçuşun seçilmesi garanti edilmez.
- Bildirim akışları yapılandırılmış e-posta bağlantısına bağlıdır.
- `InitAllApplications.xaml`, `CloseAllApplications.xaml` ve `KillAllProcesses.xaml` içinde framework şablonundan kalan sınırlı / boş uygulama yönetimi bölümleri bulunur. Sistem hatası sonrası tarayıcıyı yeniden açma ve kapanış davranışı ayrıca doğrulanmalıdır.
- Otomasyon uçuş arama ve listeleme içindir; rezervasyon veya ödeme adımı içermez.

## Testler

`Tests/` klasöründe ayar yükleme, uygulama başlatma, işlem alma, işlem yürütme ve ana akış için REFramework test dosyaları ile bir test şablonu bulunur. Bazı doğrulamalar şablon halinde veya devre dışıdır; dosyaların varlığı başarılı test sonucu anlamına gelmez.

İlk deneme için projeye özel bir kuyruk, tek satırlık güncel girdi ve kendi e-posta bağlantınızı kullanın. Sonuç Excel'ini ve Orchestrator işlem durumunu birlikte kontrol edin. Testler de kuyruk, tarayıcı ve dosyalar üzerinde işlem yapabilir.

## GitHub'a yüklemeden önce

- `Process/ExtractPageData.xaml` içindeki kişisel alıcı / hesap adreslerini ve bağlantı kimliğini kendi paylaşım politikanıza göre düzenleyin.
- `Init/Queue_Cleaning.xaml` içindeki kişisel Orchestrator çalışma alanı yolunu paylaşılabilir bir örnekle değiştirin ve kurulum adımlarını koruyun.
- Excel dosyalarını kişisel veri içermeyen örneklerle paylaşın; çalışma kitabı özellikleri ve OneDrive konum bilgilerini de gözden geçirin.
- Çalıştırma çıktıları, hata ekran görüntüleri, günlükler ve yerel önbellekleri depoya dahil etmeyin.
- `.screenshots` gibi tasarım sırasında kullanılan görselleri çalışma anındaki hata ekran görüntülerinden ayırın; gerekli tasarım kaynaklarını kontrol etmeden kaldırmayın.
- Kullanım ve dağıtım haklarını belirlemek için uygun bir `LICENSE` dosyasını ayrıca ekleyin. Bu README bir lisans tanımlamaz.

Bu doküman mevcut proje dosyaları ve Excel yapısı incelenerek hazırlanmıştır; canlı Skyscanner, Orchestrator veya e-posta üzerinde uçtan uca çalıştırma sonucu içermez.

---

# EN

# SkyscannerREF ✈️

A **UiPath REFramework** RPA project that searches for flights on Skyscanner using criteria defined in Excel and exports the results to Excel.

Each search row is handled as a transaction in an Orchestrator queue. The robot uses departure, destination, date, and passenger details to perform a **one-way** flight search and collects up to **10 results** for each search.

## Features

- Read flight search criteria in bulk from Excel.
- Bulk upload records to an Orchestrator queue and track transaction statuses.
- Search Skyscanner's Turkish interface through Chrome.
- Parse adult and child passenger counts.
- Check that the search date falls between today and one year from today.
- Extract departure location, arrival location, airline name, and price.
- Write results to a dated Excel workbook, using a separate worksheet for each search name.
- Send email notifications when fewer than 10 results or no results are found.
- Use REFramework for logging, exception handling, and screenshots on system exceptions.

## Workflow

```text
Load Config.xlsx
        ↓
Clean queue → Read input Excel → Upload records to queue
        ↓
Open Skyscanner → Retrieve a queue transaction
        ↓
Prepare date and passenger details
        ↓
Perform a one-way search → Extract up to 10 flight results
        ↓
Write results to Excel → Update queue transaction status
        ↓
Next transaction / end process
```

## Requirements

- Windows and UiPath Studio / Robot with support for Windows-compatible projects.
- Studio version recorded in the project: `26.0.200.0`. Use a Studio environment that supports the specified dependency versions.
- Microsoft Excel desktop application.
- Google Chrome with the UiPath Chrome extension enabled.
- An Orchestrator connection with permissions to read, add, and delete queue records and update transaction statuses.
- The user's own Gmail / Google Workspace connection for email notifications.
- Internet access to Skyscanner, Orchestrator, and email services.

The project type is **Process**, the target platform is **Windows**, the expression language is **Visual Basic**, and the main entry file is `Main.xaml`. Browser automation requires an interactive Windows session.

### Package dependencies

| Package | Version |
|---|---|
| UiPath.Excel.Activities | 3.6.1 |
| UiPath.GSuite.Activities | 3.11.10 |
| UiPath.Mail.Activities | 2.11.10 |
| UiPath.System.Activities | 26.6.3 |
| UiPath.Testing.Activities | 25.10.2 |
| UiPath.UIAutomation.Activities | 26.10.3 |

Dependencies are defined in `project.json` and restored through Studio when the project is opened.

## Setup

1. Clone the repository or download it as a ZIP and extract it to a local folder.
2. Open `project.json` in UiPath Studio and wait for the packages to be restored.
3. Enable the UiPath extension for Chrome.
4. Create a dedicated test queue in Orchestrator and give the robot access to the relevant folder.
5. Update the settings in `Data/Config.xlsx` for your environment.
6. Replace **Get Queue Items → FolderPath** in `Init/Queue_Cleaning.xaml` with your own Orchestrator folder. This activity contains a hardcoded reference to the developer's personal workspace; changing the Config value alone is insufficient.
7. Configure the email activities in `Process/ExtractPageData.xaml` with your own connection and recipient address. The existing connection ID and Gmail account configuration do not automatically provide access in another environment.
8. Prepare the input file and run `Main.xaml`.

> **Before running:** During initialization, `Queue_Cleaning.xaml` deletes records returned by the query that have the `New` status in the selected queue. Use a dedicated test queue rather than a shared or production queue. `Main.xaml` also contains an activity intended to terminate the `chrome.exe` process with an all-process scope; save your open Chrome sessions and review this activity for your environment before running.

## Configuration

`Data/Config.xlsx` contains the `Settings`, `Constants`, and `Assets` worksheets.

| Setting | Default value / usage |
|---|---|
| `OrchestratorQueueName` | `REF_Queue_Skyscanner` |
| `OrchestratorQueueFolder` | Relevant Orchestrator folder; blank in the current file |
| `SkyscannerInputExcelPath` | `Data\Input\SkyscannerInput.xlsx` |
| `SkyscannerWebsiteURL` | `https://www.skyscanner.com.tr/ucak-bileti` |
| `SkyscannerOutputExcelPath` | `Data\Output\SkyscannerOutput[date].xlsx` |
| `ExScreenshotsFolderPath` | `Exceptions_Screenshots` |
| `MaxRetryNumber` | `0`; queue transaction retry settings should be reviewed in Orchestrator |
| `MaxConsecutiveSystemExceptions` | `0`; the consecutive system exception limit is disabled |

`Main.xaml` also contains the `in_OrchestratorQueueName` and `in_OrchestratorQueueFolder` arguments. However, the initial queue cleanup and upload steps run before these arguments are applied to Config. During initial setup, specify the queue details directly in `Config.xlsx` and ensure that the hardcoded `FolderPath` points to the same folder.

## Input format

Default file: `Data/Input/SkyscannerInput.xlsx`. The robot reads the workbook's **first worksheet** and uses the displayed cell values.

| Column | Description | Example |
|---|---|---|
| `SEFER` | Worksheet name in the output Excel workbook | `1. Sefer` |
| `KALKIŞ` | Departure city / airport | `Ankara` |
| `VARIŞ` | Destination city / airport | `İzmir` |
| `TARİH` | Flight date in `d.MM.yyyy` format | `7.11.2026` |
| `KİŞİ SAYISI` | Adult count and optional child count | `2 Yetişkin-1 Çocuk` |

- Keep the column names exactly as shown, including Turkish characters.
- Set the display format of Excel date cells to `d.MM.yyyy`. Update the example date based on the day you run the automation.
- The date must not be earlier than the execution day or later than one year from that day.
- Use `2 Yetişkin` for adults only and `2 Yetişkin-1 Çocuk` for searches that include children.
- The current workflow selects age **7** for all children; ages are not read from the input file.
- Keep `SEFER` values unique and compliant with Excel worksheet naming rules.
- Check the sample dates in the repository before every run.

## Output

The default output name is `SkyscannerOutput[date].xlsx`, where `[date]` is replaced with the execution date in `ddMMyyyy` format. For example:

```text
Data/Output/SkyscannerOutput06102026.xlsx
```

Results for each search are written to a worksheet named after its `SEFER` value:

| Kalkış Yeri | Varış Yeri | Firma İsmi | Fiyat |
|---|---|---|---|
| ESB | ADB | Örnek Havayolu | 3.500 TL |

The row above is an illustrative example of the output format only. The Turkish output column names and sample values are retained to match the workbook. Results with an empty airline name are filled with `1+ Firma(Aktarmalı)`. Prices are exported as the text displayed on the website.

Running again on the same day with the same `SEFER` name targets the same file and worksheet; back up the output file beforehand if you need to preserve previous results.

## Project structure

```text
SkyscannerREF/
├── Main.xaml                         # REFramework state machine
├── project.json                      # Project metadata and dependencies
├── Framework/                        # Transaction retrieval, processing, and status management
├── Init/
│   ├── Read_Excel.xaml                # Read input data
│   ├── Queue_Cleaning.xaml            # Clean pending queue records
│   ├── Queue_Loading.xaml             # Upload Excel rows to the queue
│   └── Open_Website.xaml              # Open Skyscanner
├── Process/
│   ├── DataValidationAndManipulation.xaml
│   ├── NavigateToFilterPage.xaml
│   ├── FilterPage.xaml
│   ├── ExtractPageData.xaml
│   └── Write_Excel.xaml
├── Data/
│   ├── Config.xlsx
│   ├── Input/SkyscannerInput.xlsx
│   ├── Output/
│   └── Temp/
├── Tests/                            # REFramework tests and templates
├── Documentation/                    # Framework documentation
└── Exceptions_Screenshots/            # Exception screenshots
```

## Exception handling and current limitations

- Exception branches are defined for a missing input file, an invalid date range, and no flights found.
- REFramework manages queue transaction outcomes as success / business rule exception / system exception and attempts to capture screenshots on system exceptions.
- General `Exception` handlers in some child workflows rethrow errors as `SystemException`. Therefore, a business rule exception raised inside a child workflow should not be assumed to always be marked as a Business Exception in the queue.
- Searches are designed for the Turkish interface and one-way flights. Website layout changes, cookie dialogs, or CAPTCHA may require workflow updates.
- Extraction is limited to 10 rows; it does not guarantee that all results are scanned or that the 10 cheapest flights are selected.
- Notification workflows depend on the configured email connection.
- `InitAllApplications.xaml`, `CloseAllApplications.xaml`, and `KillAllProcesses.xaml` contain limited / empty application management sections inherited from the framework template. Browser reopening after a system exception and shutdown behavior require separate verification.
- The automation searches for and lists flights; it does not include booking or payment steps.

## Tests

The `Tests/` folder contains REFramework test files for settings loading, application initialization, transaction retrieval, transaction processing, and the main workflow, along with a test template. Some assertions remain as templates or are disabled; the presence of these files does not indicate successful test execution.

For an initial trial, use a dedicated queue, a single input row with a current date, and your own email connection. Check both the output Excel workbook and the Orchestrator transaction status. Tests can also act on queues, the browser, and files.

## Before uploading to GitHub

- Review and update personal recipient / account addresses and the connection ID in `Process/ExtractPageData.xaml` according to your sharing policy.
- Replace the personal Orchestrator workspace path in `Init/Queue_Cleaning.xaml` with a shareable example and retain the setup instructions.
- Share Excel files containing sample data without personal information; also review workbook properties and OneDrive location metadata.
- Exclude execution outputs, exception screenshots, logs, and local caches from the repository.
- Distinguish design-time images such as `.screenshots` from runtime exception screenshots; do not remove required design resources without checking them.
- Add an appropriate `LICENSE` file separately to define usage and distribution rights. This README does not define a license.

This document was prepared by inspecting the current project files and Excel structure; it does not report an end-to-end run against live Skyscanner, Orchestrator, or email services.
