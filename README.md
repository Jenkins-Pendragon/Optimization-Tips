# Optimization Tips

* Buildinizi Mono'dan **IL2CPP** ye çekin, Player Settings / Other Settings'den bu ayarı düzenleyebilirsiniz. Bu işlem build alırken kodumuzu C++'a çevirmektedir, bu da bedava performans demektir.


* Build almadan önce tüm **Debug.Log()** 'ları temizleyiniz. **Debug.Log()** bellekte yer tutar ve GC'nin çalışmasına sebebiyet verir.


* Vector3.magnitude & Vector3.Distance() metotları karekök işlemi kullanarak sonuç verdiğinden dolayı aşırı yorucudur. Kullanıcıya dönen değeri göstermeniz gerekmiyorsa eğer yapınızı sqrMagnitude üzerinden çalışacak şekilde yeniden düzenleyin.


* Collectionlar üzerinde işlem yaparken foreach kullanmaktan kaçının. Foreach arka planda yeni bir IEnumerator oluşturduğundan dolayı yine Garbage Collector'umuza yeni işler düşer. Bunun yerine for döngüsü kullanın.


* **Instantiate** & **new** Keyword'ü ile yarattığımız nesneler hafızada yer kaplar, dolayısıyla RunTime'da sürekli Array/Obje vs oluşturaktan kaçının. Bunun yerine başlangıçta oluşturup onu kullanın veya **Object Pooling** kullanın. (new Vector3 heap'ta yer kaplar, istediğiniz kadar kullanabilirsiniz.)


* Scriptlerinizde içi boş durum fonksiyonu bırakmayın. (Start, Update, OnEnable...) Bu metotlar boş olsa bile derlenip çalıştırılır.
```C#
    // Start is called before the first frame update
    void Start()
    {

    }

    // Update is called once per frame
    void Update()
    {
        
    }
```


* GetComponent<>() işlemini aynı scope içinde birden fazla kullanıyorsanız, bunun yerine bir kere değişkene atayın ve değişken üzerinden kullanın. Update içinde kesinlikle kullanmayın, kullanmaktan başka çareniz yoksa yine değişkene referans alarak kullanın, bir boolean kullanarak referans aldığımız satıra bir kez girmesini sağlayın. 
``` C#
public class OptimizationTips : MonoBehaviour
{
    void Update()
    {
        GetComponent<Rigidbody>().AddForce(Vector3.forward * 25.0f);
        GetComponent<Rigidbody>().AddTorque(Vector3.up * 25.0f);
    }
}
// Bunun Yerine;
public class OptimizationTips : MonoBehaviour
{
    private Rigidbody rigid;
    public void Start()
    {
        rigid = GetComponent<Rigidbody>();
    }
    public void Update()
    {
        rigid.AddForce(Vector3.forward * 25.0f);
        rigid.AddTorque(Vector3.up * 25.0f);
    }
}
```


* GameObject.Find / FindObjectOfType metotları'da yavaş işlemlerdir. FindWithTag daha tercih edilebilir ve optimizedir. Update için kullanmaktan kaçının. Start veya Awake'de kullanıyorsanız ve oyununuz ilk saniyelerde kasıyor ise, atama işlemlerini OnValidate()'e taşıyabilirsiniz.


* **GetComponents** veya **GetComponentsInChildren** fonksiyonları, belli bir türdeki tüm component’leri bulup bir array olarak döndürürler. Yeni arrayler, GC için  daha fazla iş demektir. Bunun yerine, bu fonksiyonların List parametre alan versiyonlarını kullanın. Bu şekilde, bulunan sonuçlar halihazırda var olan List’te depolanır, yeni bir array oluşmamış olur.
```C#
public List<Collider> colliders = new List<Collider>();
public void Start()
{
    GetComponentsInChildren(colliders);
    for (int i = 0; i < colliders.Count; i++)
    {
        Collider x = colliders[i];
        x.enabled = false;
    }
}
 ```
 
 
* .tag yerine CompareTag(); kullanın.
```C#
 if(obje.CompareTag("Player")) 
```

* LINQ kullanmayın. Gereken neyse kendiniz el ile yazın.


* MeshRenderer veya SkinnedMeshRenderer 'a sahip objelerin materyallerine erişip düzenlemek istediğinizde, .sharedMaterial kullanın, bu materyale doğrudan erişim sağlar. Eğer aynı materyale sahip birden çok objeniz var ve siz sadece içlerinden bir tanesini düzenlemek istiyorsanız .material kullanın, bu yöntem üzerindeki materialın kopyasını çıkartarak çalışır.


* Bir listeyi for ile dönüyorsanız ve listenin uzunluğu for döngüsü boyunca değişmeyecekse, list.Count'u bir değişkene alıp kullanın her stepte çağırılmasını engelleyin veya for döngüsünü counttan 0'a doğru ilerleyecek şekilde düzenleyin.

```C#
public List<GameObject> gameObjects = new List<GameObject>();
public void Start()
{
    for (int i = 0; i < gameObjects.Count; i++)
    {
        gameObjects[i].SetActive(i % 2 == 0);
    }
}
```
Optimize Edilmiş Kullanımı
```C#
public List<GameObject> gameObjects = new List<GameObject>();
public void Start()
{
    int count = gameObjects.Count;
    for (int i = 0; i < count; i++)
    {
        gameObjects[i].SetActive(i % 2 == 0);
    }
    // VEYA
    for (int i = gameObjects.Count - 1; i >= 0; i--)
    {
        gameObjects[i].SetActive(i % 2 == 0);
    }
}
```

* System.GC.Collect(); ile GC'yi el ile çalıştırabilirsiniz. Oyun esnasında çalıştırılmasından kaçınmak için, Loading ekranı gibi fırsatlarda kullanabilirsiniz. 


* Rigidbody / Collider'a sahip objelerinizden hareket etmeyeceğine emin olduklarınızın veya doğal kuvvetlerden etkilenmemesini istediklerinizin isKinematic değerini true yapın. Aksi takdirde fizik motoru hesaplamalar yapmaya devam edecektir.


* Edit -> Project Settings -> Physics‘teki **Auto Sync Transforms**‘u kapatın. Bu değer açık olduğunda, bir objenin Transform’u ne zaman değişirse fizik motoru bu değişikliği anında fizik dünyasına uygularken, bu değeri kapatırsanız tüm Transform değişiklikleri FixedUpdate’ten hemen önce toplu bir şekilde fizik dünyasına uygulanır. Oyununuzda çok fazla Rigidbody varsa ve Update’te sürekli Transform değerlerini değiştiriyorsanız, bu optimizasyonun performansa büyük etkisi olabilir.


* Texture ve IMG leri mümkün mertebe 4 ün katı olucak şekilde bulun, tasarlayın ve kullanın. Unity build alırken bu çözünürlüğü sıkıştırarak APK boyutunuzu düşürür. Örneğin 1024x1024 (3 mb) bir görseli 256x256 (100 kb) gibi bir duruma sıkıştırıp kullanabilirsiniz fakat 1024x1023 olması durumunda (4'ün katı olmadığı için) Compress işlemi uygulanamaz.


* Yeni oluşturulan materyallere otomatik olarak atanan Standard Shader‘ı minimum düzeyde kullanın (mobil oyunlarda hiç kullanmamaya çalışın). Bu shader, PBR adı verilen gerçekçi ışıklandırmaya yönelik hesaplamalar üzerine kurulduğu için, özellikle mobil platformlarda çok fazla performans harcar. Materyaliniz için bir shader seçerken öncelikle Mobile altında listelenmiş shader’lara bakın, burada ihtiyacınızı karşılayan bir shader yoksa o zaman Legacy Shaders‘a yönelin.


* (Özellikle mobil) Eğer sahnenizde aynı materyali taşıyan ve low-poly olan çok fazla obje varsa, Player Settings‘teki Dynamic Batching‘i açarak “Batches“ın azalmasına yardımcı olabilirsiniz. Bu ayar, Unity’nin kameranın görüş alanındaki aynı materyale sahip objeleri GPU’ya yollamadan önce birleştirerek tek bir obje yapmasını sağlar, böylece birden çok obje tek bir seferde çizilir. Bir frame’de kaç objenin birleştirildiğini, Stats ekranında “Saved by batching” olarak görebilirsiniz.


* Fog (sis) kullanmak da performans için iyi değildir. Eğer fog, oyununuzda çok ciddi bir yere sahip değilse kullanmaktan kaçının.


* Eğer gölge kullanıyorsanız, Edit-Project Settings-Quality‘den Shadow Distance‘ı olabildiğince düşürün. Bu değer, gölgelerin çizileceği en uzak mesafeyi belirler. Daha uzaktaki objelerin gölgeleri çizilmez. Quality ayarlarındaki Shadow Cascades‘i de, görsele çok büyük etkisi olmadığı sürece No Cascades yapın.


*
