The constructor is a special function that is executed during the creation of the contract and cannot be called afterwards. In this case, it permanently stores the address of the person creating the contract. The msg variable (together with tx and block) is a special global variable that contains properties which allow access to the blockchain. msg.sender is always the address where the current (external) function call came from.

Special Global Variables and Functions
https://docs.soliditylang.org/en/latest/units-and-global-variables.html#special-variables-functions


Blockchain Basis


Transactions

Blockchain, global olarak paylaşılan, işlemsel bir veritabanıdır. Herkes sadece ağa katılarak bile  veritabanındaki hareketliliği görebilir. Veritabanındaki bir şeyi değiştirmek istiyorsanız, diğerleri tarafından kabul edilmesi gereken sözde bir işlem oluşturmanız gerekir. İşlem kelimesi, yapmak istediğiniz değişikliğin (aynı anda iki değeri değiştirmek istediğinizi varsayalım) ya hiç yapılmamasını ya da tamamen uygulandığını ima eder. Ayrıca, işleminiz veritabanına uygulanırken başka hiçbir işlem onu değiştiremez.

Örnek olarak, elektronik para birimindeki tüm hesapların bakiyelerini listeleyen bir tablo hayal edin. Bir hesaptan diğerine transfer talep edilirse, veritabanının işlemsel doğası, tutar bir hesaptan çıkarıldığında her zaman diğer hesaba eklenmesini sağlar. Herhangi bir nedenle tutarın hedef hesaba eklenmesi mümkün değilse, kaynak hesapta da değişiklik yapılmaz.

Ayrıca, bir işlem her zaman gönderen (yaratıcı) tarafından şifreli olarak imzalanır. Bu, veritabanındaki belirli değişikliklere erişimi korumayı kolaylaştırır. Kripto para birimi örneğinde, basit bir kontrol, yalnızca anahtarları hesaba katan bir kişinin hesaptan para aktarabilmesini sağlar.

Bloklar

Üstesinden gelinmesi gereken en büyük engellerden biri (Bitcoin açısından) "çifte harcama saldırısı" olarak adlandırılan bir olaydır: Ağda bir cüzdanı boşaltmak isteyen eşzamanlı iki işlem varsa ne olur? İşlemlerden sadece biri geçerli olabilir, tipik olarak önce kabul edilmiş olanı. Sorun, “ilk” in eşler arası ağda nesnel bir terim olmamasıdır.

Buna cevap verilecek olursa, hiçbir kullanıcının bu konuda endişelenmesine gerek yoktur. Anlaşmazlığın çözümü sırasında kullanıcı için global olarak kabul edilen bir işlem sırası verilir. İşlemler, “blok” olarak adlandırılan birbirine bağlı bir pakette toplanır. Daha sonra tüm katılımcı düğümler ile  eşit şekilde dağıtılır. İki işlem birbiriyle çelişirse, ikinci olan reddedilir ve bloğun parçası olmaz.

Bu bloklar zaman içinde lineer bir dizi oluşturur ve buna “blockchain” denir. Zincire düzenli aralıklarla bloklar eklenir - Ethereum için her 17 saniyede yeni blok eklenir.

“Order Selection Mechanism” (madencilik) bir parçası olarak, bloklar zaman zaman geri döndürülebilir, ancak yalnızca zincirin “ucunda” olanlar bu şekilde bir işlem için elverişlidirler. Belirli bir bloğun üstüne ne kadar fazla blok eklenirse, bu bloğun geri döndürülme olasılığı o kadar düşük olur. Bu nedenle, işlemleriniz geri çevrilir ve hatta blok zincirinden kaldırılır, ancak ne kadar uzun süre beklerseniz, işleminizin geri çevrilme ihtimali de o kadar az olacaktır.

Not: İşlemlerin bir sonraki bloğa veya bir sonraki gelecek olan bloğa dahil edilmesi garanti edilmez. Çünkü bir işlemin göndericisine değil, işlemin hangi bloğa dahil edileceğini belirlemek madencilere bağlıdır.

Sözleşmenizin gelecekteki aramalarını planlamak istiyorsanız, bir akıllı sözleşme otomasyon aracı veya bir oracle hizmeti kullanabilirsiniz.


The Ethereum Virtual Machine

Genel Bakış

Ethereum Sanal Makinesi(EVM), Ethereum'da akıllı sözleşmeler için kullanılan çalışma ortamıdır. Bu alan yalnızca korunaklı değil, aynı zamanda tamamen izoledir. Yani EVM içinde çalışan bir kodun ağa, herhangi bir dosya sistemine veya diğer dış işlemlere erişimi yoktur. Burada, akıllı sözleşmelerin diğer akıllı sözleşmelere bile sınırlı erişimi vardır.

Hesaplar

Ethereum'da aynı adres alanını paylaşan iki tür hesap vardır: Public-private anahtar çiftleri (yani insanlar) ile kontrol edilen dış hesaplar ve hesapla birlikte saklanan kod tarafından kontrol edilen sözleşme hesapları.

Harici bir hesabın adresi açık anahtardan belirlenirken, bir sözleşmenin adresi sözleşmenin oluşturulduğu anda belirlenir (oluşturanın adresinden ve bu adresten gönderilen işlem sayısından türetilir, bunlara "nonce” da denir).

Hesabın kodu depolayıp saklamadığına bakılmaksızın, iki tür EVM tarafından eşit olarak ele alınır.

Her hesabın, depolama adı verilen 256-bit sözcükleri 256-bit sözcüklere eşleyen kalıcı bir anahtar-değer deposu vardır. Dahası, her hesabın Ether değerinde bir bakiyesi vardır ve bu bakiye değeri işlemler sonucunda yine Ether cinsinde değişime uğrar.

İşlemler

İşlem, bir hesaptan diğerine gönderilen bir mesajdır (aynı veya boş olabilir, aşağıya bakın). Bu mesaj, ikili verileri (“yük” olarak adlandırılır) ve Ether içerebilir.

Hedef hesap kod içeriyorsa, bu kod yürütülür ve sonucunda elde edilen veri yükü, giriş verileri olarak kabul edilir.

Hedef hesap belirtilmemişse (işlemin alıcısı yoksa veya alıcı null olarak ayarlanırsa), işlem yeni bir sözleşme oluşturmak amacını taşıyor demektir. Daha önce de belirtildiği gibi, bu sözleşmenin adresi sıfır adres değil, gönderenden ve gönderilen işlem sayısından(“nonce”) türetilen bir adrestir. Böyle bir sözleşme oluşturma işleminin yükü, EVM bayt kodu olarak alınır ve yürütülür.

Bu uygulamanın çıktı verileri, sözleşmenin kodu olarak kalıcı olarak saklanır. Aslında, bir sözleşme oluşturmak için sözleşmenin gerçek kodunu değil, bu kod çalıştırıldığında ortaya çıkan kodu gönderdiğiniz anlamına gelir.

Not: Bir sözleşme oluşturulurken kodu hala aktif değildir. Bu nedenle,geliştirici sözleşmeyi bitirene kadar sözleşme çağrı yapılmamalıdır.

Gaz (gas)

İşlemi gerçekleştirmek için gereken iş miktarını sınırlandırmak ve bu işlemin komisyonunu aynı anda ödemek amacı ile her işlem bir gaz miktarı ücretlendirilir. EVM işlemi gerçekleştirirken, gaz belirli kurallara göre kademeli olarak tüketilir.

Gaz fiyatı: (GAS_PRICE * GAS) Ön ödeme yapmak zorunda olan ve işlemi oluşturan hesap tarafından belirlenen bir değerdir. İşlem tamamlandıktan sonra bir miktar gaz kalırsa, aynı şekilde işlemi oluşturan hesaba iade edilir.

İşlem sırasında gaz biterse negatife döner ve hemen out-of-gas hatası tetiklenir. Böylece işlem için yapılan tüm değişiklikler geri alınır.


Depolama(Storage), Bellek(Memory) & Yığın(Stack)

EVM, veri depolama, bellek ve yığından oluşan üç depolama alanına sahiptir.

Her hesabın, metot çağrıları ve işlemler arasında kalıcı olan, depolama adı verilen bir veri alanı vardır. Depolama, 256 bit sözcükleri 256 bit sözcüklerle eşleyen bir anahtar-değer deposudur. Bir sözleşmeden depolamayı belirtmek mümkün değildir, okuması nispeten pahalı bir işlemken, depolamayı başlatmak ve değiştirmek daha da fazla maliyetlere sebep olacaktır. 

Bu maliyet nedeniyle, kalıcı depolamada depoladıklarınızı (sözleşmenin çalışması için gerekenler) en aza indirmelisiniz. Türetilmiş hesaplamalar, önbelleğe alma ve toplamalar gibi verileri sözleşmenin dışında saklayın. Bir sözleşme, kendisinin dışında herhangi bir depoya ne okuyabilir ne de yazabilir.
