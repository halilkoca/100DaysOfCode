
Depolama(Storage), Bellek(Memory) & Yığın(Stack)

EVM, veri depolama, bellek ve yığından oluşan üç depolama alanına sahiptir.

Her hesabın, metot çağrıları ve işlemler arasında kalıcı olan, DEPOLAMA adı verilen bir veri alanı vardır. Depolama, 256 bit sözcükleri 256 bit sözcüklerle eşleyen bir anahtar-değer deposudur. Bir sözleşmeden depolamayı belirtmek mümkün değildir, okuması nispeten pahalı bir işlemken, depolamayı başlatmak ve değiştirmek daha da fazla maliyetleri beraberinde getirir. Bu maliyet nedeni ile kalıcı depolama alanında sözleşmenin çalışması için yeterli düzeyde en az veri saklayarak ilerlenmelidir. Türetilmiş hesaplamalar, önbelleğe alma ve toplamalar(aggregates) gibi verileri sözleşmenin dışında saklayın. Bir sözleşme, kendisinin dışında herhangi bir depoya ne okuyabilir ne de yazamaz.

İkinci veri alanı, bir sözleşmenin her yeni işlem çağrısı için temiz bir instance oluşturduğu alana BELLEK denir. Bellek doğrusaldır ve bayt düzeyinde adreslenebilir, ancak okumalar 256 bitlik bir genişlikle sınırlıdır, yazmalar ise 8 bit veya 256 bit genişliğinde olabilir. Daha önce dokunulmamış bir bellek alanına erişirken(okuma-yazma) hafıza alanı (256-bit) genişletilir. Genişleme yapılırken gaz bedeli ödenmelidir. Bellek büyüdükçe daha maliyetli olur (artış karesi ile hesaplanır).

EVM bir kayıt makinesi değil, bir yığın makinesidir(stack machine). Bu nedenle tüm hesaplamalar yığın(stack) adı verilen bir veri alanında gerçekleştirilir. Bir yığın maksimum 1024 eleman boyutuna sahiptir ve 256 bitlik veriler içerir. Yığına erişim, aşağıdaki şekilde üst uç ile sınırlıdır: En üstteki 16 öğeden birini yığının üstüne kopyalamak veya en üstteki öğeyi altındaki 16 öğeden biriyle değiştirmek mümkündür. Diğer tüm işlemler, yığından en üstten 2 (veya işleme bağlı olarak bir veya daha fazla) öğeyi alır ve sonucu yığının en üstüne ekler. Yığının derinlerine erişim sağlamak için yığın öğelerini depolamaya veya başka bir belleğe taşımak mümkündür. Ancak yığının üstündekileri çıkartmadan yığının daha derinlerindeki rastgele öğelere erişmek mümkün değildir.


Komut Seti(Instruction Set)

EVM'nin komut seti, fikir birliği(konsensüs) sorunlarına neden olabilecek yanlış veya tutarsız uygulamalardan kaçınmak için minimum düzeyde tutulur. Tüm komutlar, temel veri türü, 256 bitlik sözcükler veya bellek dilimleri (veya diğer bayt dizileri) üzerinde çalışır. Aritmetik, bit, mantıksal ve karşılaştırma operatörleri mevcuttur. Koşullu ve koşulsuz geçişler mümkündür. Ayrıca sözleşmelerin, mevcut bloğun numarası ve zaman damgası gibi özelliklerine erişebilir.

Tam bir liste için lütfen satır içi montaj belgelerinin bir parçası olarak işlem kodlarının listesine bakın.
Link: https://docs.soliditylang.org/en/latest/yul.html#opcodes


Mesaj Çağrıları(Message Calls)

Sözleşmeler, diğer sözleşmeleri çağırabilir veya sözleşmesiz hesaplara mesaj çağrısı yoluyla Ether gönderebilir. Mesaj çağrıları, bir kaynağa, bir hedefe, veri yüküne, Eter'e, gaza ve dönüş verilerine sahip olmaları bakımından işlemlere(transactions) benzer. Aslında, her işlem, sırayla daha fazla mesaj çağrılarını oluşturabilen bir üst düzey mesaj çağrısı kümesinden oluşur.

Bir sözleşme, kalan gazının ne kadarının iç mesaj çağrısı ile gönderilmesi gerektiğine ve ne kadarını elinde tutmak istediğine karar verebilir. İç çağrıda (veya başka herhangi bir istisnada) gazın bitmesi istisnası meydana gelirse, bu durum yığına konan bir hata değeri ile bildirilecektir. Bu durumda sadece çağrı ile birlikte gönderilen gaz kullanılır. Solidity'de, çağrı sözleşmesi, bu tür durumlarda varsayılan olarak manuel bir istisnaya neden olur, böylece çağrı yığınında istisnalar zincirleme olarak çoğalmış olur.

Daha önce de söylendiği gibi, çağırılan sözleşme (çağıranla aynı olabilir) yeni temiz bir bellek örneği(instance) alır ve çağrı yüküne(payload) erişime sahip olur. Bu çağrı verileri(calldata) adı verilen ayrı bir alanda sağlanır. Uygulama işlerini tamamlayınca, çağrı üreten tarafından önceden tahsis edilen çağrı hafızasındaki bir yerde saklanan verileri döndürebilir. Tüm bu tür çağrılar tamamen senkronizedir.

Çağrılar 1024 bitlik alan ile sınırlıdır, bu da daha karmaşık işlemler için tekrarlı çağrılar yerine döngülerin tercih edilmesi gerektiği anlamına gelir. Ayrıca, bir mesaj çağrısında gazın sadece 63/64'ü iletilebilir, bu da pratikte 1000 bit'ten biraz daha az bir alan sınırına neden olur.

### Temsilci Çağrıları && Çağrı Kodu & Kütüphaneler (Delegatecall / Callcode and Libraries)

Bir mesaj çağrısı ile temelde aynı anlama gelen delegatecall, hedef adresteki kodun arama sözleşmesi bağlamında yürütülmesi ve `msg.sender` ve `msg.value` değerlerinin değiştirilememesi gibi özellikleri ile mesaj çağrısının özel bir çeşidi olarak kabul edilir. Bu, bir sözleşmenin çalışması sırasında  farklı bir adresden dinamik olarak kod yükleyebileceği anlamına gelir.

Depolama, geçerli adres ve bakiye hala çağırılan sözleşmeye atıfta bulunurken, yalnızca aranan adresten kod alınır.

Bu, Solidity'deki "library" özelliğinin uygulanmasını mümkün kılar: Bir sözleşmenin deposuna uygulanabilen yeniden kullanılabilir kitaplık kodu, ör. karmaşık bir veri yapısı uygulamak için.

### Loglar

Verileri, tümüyle blok seviyesine kadar haritalayan özel olarak indekslenmiş bir veri yapısında depolamak mümkündür. Kayıt(log) adı verilen bu özellik, event'ları gerçekleştirmek için Solidity tarafından kullanılır. Sözleşmeler, oluşturulduktan sonra günlük verilerine erişemez, ancak bu kayıtlara blok zincirinin dışından verimli bir şekilde erişilebilir. Günlük verilerinin bir kısmı bloom filtrelerinde depolandığından, bu verileri verimli ve kriptografik olarak güvenli bir şekilde aramak mümkündür. Böylece tüm blok zincirini indirmeyen ağ eşleri(hafif istemciler) bu kayıtlara erişim sağlayabilir.


### Oluşturma(Create)

Sözleşmeler, özel bir işlem kodu(opcode) kullanarak başka sözleşmeleri oluşturabilir. Bunu hedef adresi boş bırakara yapabilirler(yalnızca sıfır adresini çağırmazlar). Bu oluşturma çağrıları ile normal mesaj çağrıları arasındaki tek fark, payload verilerinin çalıştırılmasıdır. Sonucun da kod olarak saklanması ve oluşturan kişinin yığındaki yeni sözleşmenin adresini almasıdır.

###  Deactivate and Self-destruct

Kodu blok zincirinden kaldırmanın tek yolu, o adresteki bir sözleşmenin kendi kendini yok etme(selfdestruct) işlemini gerçekleştirmesidir. Bu adreste depolanan kalan Ether, belirlenmiş bir hedefe gönderilir ve ardından depolama ve kod durumdan çıkarılır. Sözleşmeyi teoride kaldırmak iyi bir fikir gibi görünüyor, ancak potansiyel olarak tehlikeli, sanki biri kaldırılmış sözleşmelere Ether gönderiyormuş gibi, Ether sonsuza kadar kaybolmuş gibi.

### [Uyarı](#)
Bir sözleşme kendi kendini imha ederek kaldırılsa bile, hala blok zincirinin tarihinin bir parçasıdır ve muhtemelen çoğu Ethereum düğümü tarafından korunur. Bu nedenle, kendi kendini yok etmeyi kullanmak, bir sabit diskten veri silmekle aynı şey değildir.

### [Not](#)
Bir sözleşmenin kodu kendi kendini yok etmeye yönelik bir çağrı içermese bile, bu işlemi yine de temsilci çağrısı(delegatecall) veya çağrı kodu kullanarak gerçekleştirebilir.


Sözleşmelerinizi devre dışı bırakmak istiyorsanız, bunun yerine tüm işlevlerin geri dönmesine neden olan bazı dahili durumları değiştirerek bunları devre dışı bırakmalısınız. Bu, Ether'i hemen iade ettiği için sözleşmeyi kullanmayı imkansız hale getirir.


### Önceden Derlenmiş Sözleşmeler(Precompiled Contracts)

Özel olan küçük bir dizi sözleşme adresi vardır: 1 ile (dahil) 8 arasındaki adres aralığı, başka herhangi bir sözleşme olarak adlandırılabilecek ancak davranışları (ve gaz tüketimleri) EVM koduyla tanımlanmayan “önceden derlenmiş sözleşmeler” içerir ve bu adreste depolanır (kod içermezler). Bunun yerine EVM yürütme ortamının kendisinde uygulanır.

Farklı EVM uyumlu zincirler, önceden derlenmiş farklı bir sözleşme seti kullanabilir. Gelecekte Ethereum ana zincirine önceden derlenmiş yeni sözleşmelerin eklenmesi de mümkün olabilir, ancak makul olarak bunların her zaman 1 ile 0xffff (dahil) aralığında olmasını bekleyebilirsiniz.