问题: index.php 实例化了$app  使用Facade 静态方法如何保证使用的是同一个实例

1  kernel \Illuminate\Foundation\Bootstrap\RegisterFacades::class

bootstarp配置引入Facade注册

2  Facade bootStarp

```php
public function bootstrap(Application $app)
    {
    //清空实例
        Facade::clearResolvedInstances();
		//设置当前$app实例
        Facade::setFacadeApplication($app);

        AliasLoader::getInstance(array_merge(
            $app->make('config')->get('app.aliases', []),
            $app->make(PackageManifest::class)->aliases()
        ))->register();
    }
   
  //绑定当前的$app 到facade定义的静态变量$app下
  public static function setFacadeApplication($app)
    {
        static::$app = $app;
    }
```



3 举例说明Route::get

```php
 class Route extends Facade
{
    /**
     * Get the registered name of the component.
     *
     * @return string
     */
     //定义返回的实例
    protected static function getFacadeAccessor()
    {
        return 'router';
    }
}

//魔术方法
public static function __callStatic($method, $args)
    {
        $instance = static::getFacadeRoot();

        if (! $instance) {
            throw new RuntimeException('A facade root has not been set.');
        }

        return $instance->$method(...$args);
    }

 //到$app解析对应的实例
 protected static function resolveFacadeInstance($name)
    {
        if (is_object($name)) {
            return $name;
        }

        if (isset(static::$resolvedInstance[$name])) {
            return static::$resolvedInstance[$name];
        }

        if (static::$app) {
            return static::$resolvedInstance[$name] = static::$app[$name];
        }
    }
```

