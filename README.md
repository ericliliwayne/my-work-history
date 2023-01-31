# 我的工作歷程
※這裡將會記錄我工作中所學所做的，不定期持續更新中...
## 前言
上了職訓局的轉職課程後，對前端HTML、CSS、Javascript以及後端PHP、SQL有基本的能力，大約一個月找到了第一份轉職的工作，這家公司是幫客戶做網站開發、SEO及在linux伺服器測試-上架，還有一些是做網站爬蟲、動畫設計、網頁設計等等工作，而我是做後端php的開發及維護的工作。以下是我在工作上所學到的及所做的事。

## 開發工具
<ol>
<li><a href="https://www.jetbrains.com/phpstorm/">PhpStorm</a></li>
<li><a href="https://code.visualstudio.com/">Visual Studio Code</a></li>
</ol>
目前主要是用這2個開發工具，PhpStorm主要是用在以php框架開發及維護使用；而Visual Studio Code則是用在前端HTML、CSS、Javascript以及原生php上。

## 使用框架
<ol>
<li><a href="https://laravel.com/">Laravel</a> 版本9</li>
<li><a href="https://codeigniter.tw/">CodeIgniter</a> 版本3</li>
</ol>
Laravel做專案開發；CodeIgniter做舊專案的維護。Laravel在MVC架構上比CodeIgniter嚴謹但語法上給我簡潔有力的感覺；CodeIgniter雖然也是MVC架構但感覺起來php語法比較偏向原生語法。

## 其他工具
<ol>
<li><a href="https://www.apachefriends.org/zh_tw/index.html">XAMPP</a> : 建置Apache、PHP及MariaDB環境的整合包，目前PHP8.1版本做開發；7.4、5.6版本作維護使用。依情況做版本切換。</li>
<li><a href="https://www.postman.com/">Postman</a> : API測試平台，建立好請求的URL和帶入的參數，測試是否成功送出請求及收到回傳資料是否符合規定的格式。</li>
<li><a href="https://www.sourcetreeapp.com/">Sourcetree</a> : 管理專案各分支，目前主要用來做分支merge使用。</li>
<li><a href="https://www.putty.org/">PuTTY</a> : 使用SSH連線方式連結Linux伺服器，建立測試網站，在網站正式上線前檢查是否有BUG問題、上線網站SSL更新等等...</li>
<li><a href="https://www.heidisql.com/">HeidiSQL</a> : 管理線上網站的資料庫，手動或輸入指令來做增刪查改或是備份資料庫的資料。</li>
<li><a href="https://filezilla-project.org/">FileZilla</a> : 用來更動伺服器上的檔案。</li>
<li><a href="https://gitlab.com/">GitLab</a> : 公司新舊專案放置管理的地方。</li>
</ol>

## 展示區
<ol>
<h3><strong>電子信箱驗證</strong></h3>
<ul>
使用框架:laravel 9<br>
功能需求:建立一個輸入欄位可以輸入一個或多個email，送出驗證時需各別驗證每個email是否重複及格式是否正確。<br>
* * *<br>
首先，一般email驗證會如以下這樣:

```php
    public function rules()
    {
        return [
            'email' => 'required|email',
        ];
    }
```

因為預設的驗證模式只能對單一的email字串做驗證，多筆email組成的字串無法直接使用，所以只能透過自訂規則的方式，先把字串拆開成一個一個email的陣列，才能去驗證每個email是否重複及格式是否正確。如以下程式碼所示:

```php
class Emailsvalid implements InvokableRule
{
    /**
     * Run the validation rule.
     *
     * @param  string  $attribute
     * @param  mixed  $value
     * @param  \Closure(string): \Illuminate\Translation\PotentiallyTranslatedString  $fail
     * @return void
     */
    public function __invoke($attribute, $value, $fail)
    {
        $emails = Setup::find(1)->toArray(); //抓取資料庫的email資料
        $strings['email'] = explode(',',$value); //輸入的多筆資料以「，」分開
        $strings2 = explode(',',$emails['email_address']); //資料庫的email資料以「，」分開

        foreach ($strings2 as $string2){ //資料庫的email資料分別對照輸入的資料是否重複
            if(in_array($string2,$strings['email'])){
                $fail($string2.'已經存在請重新填寫!!');
            }
        }

        //將處理好的資料作驗證
        Validator::make($strings, [
            'email' => 'required|array',
            'email.*' => 'required|email',
        ])->validate();

    }
}
```

寫好後引入到原本的Request檔案:

```php
public function rules()
    {
            return [
                'email_address' => ['required','string',new Emailsvalid],
            ];
    }
```

</ul>
<hr>
<h3><strong>資料庫塞入初始資料</strong></h3>
<ul>
測試程式時有時會塞一些資料進資料庫以利測試，若一筆一筆輸入較浪費時間，所以可以利用撰寫Seeder的方式來填充資料。<br>
使用框架:laravel 9<br>
* * *<br>

假設我想要生產1萬筆的資料可以這樣做:<br>
ArticleFactory:

```php
class ArticleFactory extends Factory
{
    /**
     * Define the model's default state.
     *
     * @return array<string, mixed>
     */
    public function definition()
    {
        return [
            'title' => fake()->name(),
            'textarea' => fake()->name(),
        ];
    }
}
```

ArticleSeeder:

```php
class ArticleSeeder extends Seeder
{
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        Article::factory()->count(10000)->create();
    }
}
```

然後執行artisan指令，不過這樣還無法成功執行，因為要先在DatabaseSeeder檔案中 call ArticleSeeder 才能成功執行:

```php
class DatabaseSeeder extends Seeder
{
    public function run()
    {
        $this->call([
            ArticleSeeder::class,
        ]);
    }
}
```

</ul>
<hr>
<h3><strong>產生API</strong></h3>
<ul>
使用框架:laravel 9<br>
功能需求:。<br>
</ul>
<h3><strong>第三方登入</strong></h3>
<ul>* Google</ul>
<ul>* Line</ul>


    






