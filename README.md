<p align="center"><img src="https://despark.com/public/images/despark-logo.svg"></p>

<p align="center">
<a href="https://packagist.org/packages/despark/laravel-db-i18n"><img src="https://poser.pugx.org/despark/laravel-db-i18n/v/stable.svg" alt="Latest Stable Version"></a>
</p>

# Despark's igniCMS DB Localization Module
## About
This package extends [despark/igni-core](https://github.com/despark/igni-core) by adding a fully functional DB Localization Module.

## Installation
Require using [Composer](https://getcomposer.org)
```bash
composer require despark/laravel-db-i18n
```

## Example usage
> config/ignicms.php
```php
   ...
    'languages' => [
        // Add languages that you will use in your app.
        [
            'locale' => 'en',
            'name' => 'English',
        ],
        [
            'locale' => 'de',
            'name' => 'Deutsche',
        ],
        [
            'locale' => 'fr',
            'name' => 'Français',
        ],
    ],
   ...
  ```
> database/migrations/create_articles_table.php

  ```php
   ...
    /**
     * Run the migrations.
     *
     * @return void
     */    
    public function up()
    {
        Schema::create('articles', function (Blueprint $table) {
            $table->increments('id');
            $table->string('url');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('articles');
    }
   ...
  ```
> database/migrations/create_articles_i18n_table.php

  ```php
   ...
    /**
     * Run the migrations.
     *
     * @return void
     */    
    public function up()
    {
        Schema::create('articles_i18n', function (Blueprint $table) {
            $table->increments('id');
            $table->unsignedInteger('parent_id');
            $table->string('locale');
            $table->string('title');
            $table->text('content')->nullable();
            $table->timestamps();

            $table->foreign('parent_id')
                   ->references('id')
                   ->on('articles')
                   ->onDelete('cascade')
                   ->onUpdate('cascade');
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('articles_i18n');
    }
   ...
  ```
> App\Models\Article.php

  ```php
   ...    
    use Despark\Cms\Models\AdminModel;
    use Despark\LaravelDbLocalization\Contracts\Translatable;
    use Despark\LaravelDbLocalization\Traits\HasTranslation;

    class Article extends AdminModel implements Translatable
    {
        use HasTranslation;

        protected $table = 'articles';

        protected $translatable = [
            'title',
            'content',
        ];

        protected $fillable = [
            'title',
            'content',
            'url',
        ];

        protected $rules = [
            'title' => 'required',
            'content' => 'required',
            'url' => 'required',
        ];

        protected $identifier = 'articles';
     }
   ...
  ```

> App\Http\Controllers\Admin\ArticlesController.php

   ```php
   ...    
    use Despark\Cms\Http\Controllers\AdminController;

    class ArticlesController extends AdminController
    {
    }
   ...
  ```
> config\entities\articles.php
```php
   return [
        'name' => 'Articles',
        'description' => 'Articles resource',
        'model' => App\Models\Article::class,
        'controller' => App\Http\Controllers\Admin\ArticlesController::class,
        'adminColumns' => [
            'title' => 'translation.title',
            'created at' => 'created_at',
        ],
        'actions' => ['edit', 'create', 'destroy'],
        'adminFormFields' => [
            'title' => [
                'type' => 'text',
                'label' => 'Title',
            ],
            'content' => [
                'type' => 'textarea',
                'label' => 'Content',
            ],
            'url' => [
                'type' => 'text',
                'label' => 'Url',
            ],
        ],
        'adminMenu' => [
            'articles' => [
                'name' => 'Articles',
                'iconClass' => 'fa-newspaper-o',
                'link' => 'articles.index',
            ],
        ],
    ];
  ```

