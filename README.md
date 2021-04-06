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
<img width="449" alt="Ekran Resmi 2021-04-06 14 19 30" src="https://user-images.githubusercontent.com/75368035/113703159-1e002280-96e3-11eb-90a3-4e95c8b03333.png">

* .tag yerine CompareTag(); kullanın.
```C#
 if(obje.CompareTag("Player")) 
```
