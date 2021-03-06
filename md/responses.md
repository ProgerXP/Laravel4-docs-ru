# Шаблоны и отклики

- [Базовые отклики](#basic-responses)
- [Переадресация](#redirects)
- [Шаблоны](#views)
- [Составители](#view-composers)
- [Особые отклики](#special-responses)

<a name="basic-responses"></a>
## Базовые отклики

**Возврат строк из маршрутов:**

	Route::get('/', function()
	{
		return 'Привет, мир!';
	});

**Создание собственного ответа (отклика)**

Объект `Response` наследует класс `Symfony\Component\HttpFoundation\Response` который предоставляет набор методов для построения отклика HTTP.

	$response = Response::make($contents, $statusCode);

	$response->header('Content-Type', $value);

	return $response;

**Добавление cookie к ответу:**

	$cookie = Cookie::make('name', 'значение');

	return Response::make($content)->withCookie($cookie);

<a name="redirects"></a>
## Переадресация

**Returning A Redirect**

	return Redirect::to('user/login');

**Переадресация с одноразовыми переменными сессии:**
	
	return Redirect::to('user/login')->with('message', 'Войти не удалось');

> **Примечание:** Метод `with` сохраняет данные в сессии, поэтому вы можете прочитать их, используя обычный метод `Session::get`.

**Переадресация на именованный маршрут:**

	return Redirect::route('login');

**Переадресация на именованный маршрут с параметрами:**

	return Redirect::route('profile', array(1));

**Переадресация на именованный маршрут с именованными параметрами:**

	return Redirect::route('profile', array('user' => 1));

**Переадресация на действие контроллера**

	return Redirect::action('HomeController@index');

**Переадресация на действие контроллера с параметрами:**

	return Redirect::action('UserController@profile', array(1));

**Переадресация на действие контроллера с именованными параметрами:**

	return Redirect::action('UserController@profile', array('user' => 1));

<a name="views"></a>
## Шаблоны

Шаблоны (views) обычно содержат HTML-код вашего приложения и представляют собой удобный способ разделения логики контроллеров и обработки страниц от их представления. Шаблоны хранятся в папке `app/views`.

Простой шаблон может иметь такой вид:

	<!-- Шаблон из файла app/views/greeting.php -->

	<html>
		<body>
			<h1>Привет, <?php echo $name; ?></h1>
		</body>
	</html>

Этот шаблон может быть возвращён в браузер таким образом:

	Route::get('/', function()
	{
		return View::make('greeting', array('name' => 'Тейлор'));
	});

Второй параметр, переданный в `View::make` - массив данных, которые должны быть доступны внутри шаблона.

**Передача переменных в шаблон:**

	$view = View::make('greeting')->with('name', 'Стив');

В примере выше переменная `$name` будет доступна в шаблоне и будет иметь значение `Steve`.

Можно передать массив данных в виде второго параметра для метода `make`:

	$view = View::make('greetings', $data);

Вы также можете "поделить" данные между всеми шаблонами, сделав их глобальной:

	View::share('name', 'Стив');

**Передача вложенного шаблона в шаблон**

Иногда вам может быть нужно передать шаблон внутрь другого шаблона. Например, имея шаблон `app/views/child/view.php` нам может понадобиться передать его в другой шаблон и мы можем сделать это так:

	$view = View::make('greeting')->nest('child', 'child.view');

	$view = View::make('greeting')->nest('child', 'child.view', $data);

Затем вложенный шаблон может быть отображён в родительском шаблоне:

	<html>
		<body>
			<h1>Привет!</h1>
			<?php echo $child; ?>
		</body>
	</html>

<a name="view-composers"></a>
## Составители

Составители шаблонов - функции обратного вызова или методы класса, которые вызываются, когда данный шаблон формируется (отображается) в строку. Если у вас есть данные, которые вы хотите привязать к шаблону при каждом его формировании, то составители помогут вам выделить такую логику в единое место. Таким образом, составители могут работать как "модели шаблонов" или "представители".

**Регистрация составителя:**

	View::composer('profile', function($view)
	{
		$view->with('count', User::count());
	});

Теперь при каждом отображении шаблона `profile` к нему будет привязана переменная `count`.

Вы можете привязать составитель сразу к неским шаблонам:

    View::composer(array('profile','dashboard'), function($view)
    {
        $view->with('count', User::count());
    });

Если вам больше нравятся методы классов, что позволяет вам использовать регистрации в [IoC Container](/docs/ioc), то вы можете сделать это так:

	View::composer('profile', 'ProfileComposer');

Класс составителя должен иметь такой вид:

	class ProfileComposer {

		public function compose($view)
		{
			$view->with('count', User::count());
		}

	}

Обратите внимание, что нет строгого правила, где должны храниться классы-составители. Вы можете поместить их в любое место, где их сможет найти автозагрузчик в соответствии с директивами в вашем файл `composer.json`.

### Создатели

Создатели шаблонов работают почти так же, как **составители** , но вызываются сразу после создания объекта шаблона, а не во время его отображения в строку.Для регистрации нового создателя используйте метод `creator`:

	View::creator('profile', function($view)
	{
		$view->with('count', User::count());
	});

<a name="special-responses"></a>
## Особые отклики

**Создание JSON-ответа:**

	return Response::json(array('name' => 'Стив', 'state' => 'CA'));

**Создание JSONP-ответа:**

	return Response::json(array('name' => 'Стив', 'state' => 'CA'))->setCallback(Input::get('callback'));

**Создание отклика загрузки файла:**

	return Response::download($pathToFile);

	return Response::download($pathToFile, $name, $headers);
