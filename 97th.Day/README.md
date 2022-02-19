###  Solidity Derleyicisini Yükleme(Installing the Solidity Compiler)

### Versiyonlama (Versioning)

Solidity versiyonları Semantic versiyonlamayı takip eder. Ek olarak, ana sürüm 0'a (yani 0.x.y) sahip yama düzeyindeki sürümler, büyük değişiklikleri içermeyecektir. Bu da, 0.x.y sürümüyle derlenen kodun, z > y olduğu yerde 0.x.z ile derlenmesi anlamına gelir.

Sürümlere ek olarak, geliştiricilerin gelecek özellikleri denemelerini ve erken geri bildirim sağlamalarını kolaylaştırmak amacıyla her gece geliştirme derlemeleri(nightly development builds) sağlıyoruz. Bununla birlikte, gece derlemelerinin genellikle çok kararlı olmasına rağmen, geliştirme dalından son teknoloji kodlar içerdiğini ve her zaman çalışacakları garanti edilmediğini unutmayın. En iyi çabalarımıza rağmen, gerçek bir sürümün parçası olmayacak belgelenmemiş ve/veya bozuk değişiklikler içerebilirler. Üretim amaçlı kullanılmazlar.

Sözleşmeleri dağıtırken, Solidity'nin en son yayınlanan sürümünü kullanmalısınız. Bunun nedeni, son zamanlarda yapılan değişikliklerin yanı sıra yeni özelliklerin ve hata düzeltmelerinin düzenli olarak sunulmasıdır. Bu hızlı değişim hızını belirtmek için şu anda bir 0.x sürüm numarası kullanıyoruz.


### Remix

Küçük sözleşmeler ve Solidity'yi hızlı bir şekilde öğrenmek için Remix'i öneriyoruz.

Remix'e çevrimiçi erişilebilir ve hiçbir şey yüklemeniz gerekmez. İnternet bağlantısı olmadan kullanmak istiyorsanız, https://github.com/ethereum/remix-live/tree/gh-pages adresine gidin ve o sayfada açıklandığı gibi .zip dosyasını indirin. Remix, aynı zamanda, birden fazla Solidity sürümü yüklemeden gece yapılarını test etmek için uygun bir seçenektir.

Bu sayfadaki diğer seçenekler, komut satırı Solidity derleyici yazılımının bilgisayarınıza yüklenmesiyle ilgili ayrıntıları içerir. Daha büyük bir sözleşme üzerinde çalışıyorsanız veya daha fazla derleme seçeneğine ihtiyacınız varsa, bir komut satırı derleyici seçin.

### npm / Node.js

npm install -g solc


### Docker

docker run ethereum/solc:stable --help


docker run -v /local/path:/sources ethereum/solc:stable -o /sources/output --abi --bin /sources/Contract.sol


### Github

git clone --recursive https://github.com/ethereum/solidity.git
cd solidity