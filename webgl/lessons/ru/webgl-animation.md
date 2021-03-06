Title: WebGL - Анимация
Description: Анимация в WebGL

Эта статья продолжает серию статей о WebGL. Первая была об
<a href="webgl-fundamentals.html">основах</a>, а в предыдущей мы рассмотрели
<a href="webgl-3d-camera.html">камеры</a>. Если вы их ещё не читали, то
рекомендую сначала ознакомиться с ними.

Как создать анимацию в WebGL?

На самом деле анимация - это больше работа JavaScript, нежели WebGL.
Для анимации необходимо менять что-либо в JavaScript с течением времени
и при этом постоянно вызывать отрисовку.

Мы можем взять один из наших предыдущих примеров и сделать анимацию:

    *var fieldOfViewRadians = degToRad(60);
    *var rotationSpeed = 1.2;

    *requestAnimationFrame(drawScene);

    // отрисовка сцены
    function drawScene() {
    *  // каждый кадр немного увеличивается поворот
    *  rotation[1] += rotationSpeed / 60.0;

      ...
    *  // в следующем кадре снова вызываем drawScene
    *  requestAnimationFrame(drawScene);
    }

И анимация готова:

{{{example url="../webgl-animation-not-frame-rate-independent.html" }}}

Однако, здесь есть одна неявная проблема. В коде выше содержится
`rotationSpeed / 60.0`. Мы делим на 60.0, потому что предполагаем, что браузер
вызовет requestAnimationFrame 60 раз за секунду, что является близким к правде.

Однако, это не верное предположение. Возможно, у пользователя медленное
устройство вроде старого смартфона. Или, возможно, у пользователя в фоне запущена
очень ресурсоёмкая программа. Есть множество причин, по которым браузер может не
отображать 60 кадров в секунду. Возможно, наступил 2020 год и все устройства
работают на частоте 240 кадров в секунду. А возможно, пользователь большой фанат
игр и у него ЭЛТ-монитор с частотой 90 кадров в секунду.

Проблема наглядно продемонстрирована на следующем примере:

{{{diagram url="../webgl-animation-frame-rate-issues.html" }}}

В примере выше мы хотели бы поворачивать все буквы 'F' с одной скоростью. Буква
'F' посередине работает на полной скорости и её частота кадров ни от чего не
зависит. Буквы слева и справа симулируют, будто браузер работает на 1/8 от полной
мощности компьютера. Буква слева **ЗАВИСИМА** от частоты кадров. Буква справа
**НЕЗАВИСИМА** от частоты кадров

Заметьте, что буква слева не принимает во внимание, что частота кадров может быть
низкой, и не успевает поворачиваться. Буква справа, даже несмотря на работу в
режиме 1/8 доли от частоты кадров, успевает вращаться и не отстаёт от средней
буквы, вращающейся на полной скорости.

Для того, чтобы сделать анимацию независимой от кадров, необходимо вычислять,
сколько времени проходит между кадрами, и использовать это для определения
времени анимации текущего кадра.

Для начала нам нужно получить время. К счастью, `requestAnimationFrame` при своём
вызове передаёт нам время, прошедшее с загрузки страницы.

Лично мне кажется, что проще всего работать со временем в секундах, но так как
`requestAnimationFrame` передаёт время в миллисекундах (тысяча секунд), нам нужно
умножить на 0.001, чтобы получить секунды.

После этого мы сможем получить разницу во времени.

    *var then = 0;

    requestAnimationFrame(drawScene);

    // отрисовка сцены
    *function drawScene(now) {
    *  // получаем время в секундах
    *  now *= 0.001;
    *  // вычитаем предыдущее значение времени от текущего
    *  var deltaTime = now - then;
    *  // запоминаем текущее время для следующего кадра
    *  then = now;

       ...

Когда у нас есть `deltaTime` в секундах, наши расчёты будут строиться на том,
сколько раз в секунду мы хотим выполнять что-либо. В нашем случае значение
`rotationSpeed` равно 1.2, то есть мы хотим выполнять поворот на 1.2 радиана
в секунду. Это примерно 1/5 поворота или, другими словами, нам необходимо 5
секунд для полного поворота независимо от частоты кадров.

    *    rotation[1] += rotationSpeed * deltaTime;

Вот рабочий пример.

{{{example url="../webgl-animation.html" }}}

Вероятно, вы не заметите разницы с примером в верху страницы, если только
у вас не медленный компьютер. Но если не учитывать частоту кадров, у
некоторых ваших пользователей картина может сильно отличаться от той,
которую вы планировали.

Далее рассмотрим <a href="webgl-3d-textures.html">работу с текстурами</a>.

<div class="webgl_bottombar">
<h3>Не используйте функции setInterval и setTimeout!</h3>
<p>Если в прошлом вам приходилось создавать анимацию в JavaScript, возможно, вы
использовали либо <code>setInterval</code>, либо <code>setTimeout</code> для
вызова функции отрисовки.
</p><p>
Существует две проблемы, связанные с использованием функций <code>setInterval</code>
и <code>setTimeout</code> для анимации. Первая заключается в том, что обе этих функции
никак не привязаны к механизму отрисовки браузера. Они не синхронизованы со временем,
когда браузер собирается отрисовать новый кадр, и поэтому могут потерять синхронизацию
с устройством пользователя. Если вы используете <code>setInterval</code> или
<code>setTimeout</code> и предполагаете, что будет 60 кадров в секунду, а компьютер
пользователя работает с другой частотой кадров, то вы потеряете синхронизацию с
компьютером пользователя.
</p><p>
Другой проблемой является то, что браузер понятия не имеет, почему вы используете
<code>setInterval</code> или <code>setTimeout</code>. Поэтому даже когда страницу
не видно (например, если вкладка браузера не активна), браузер по-прежнему будет
выполнять ваш код. Возможно, вы используете <code>setTimeout</code> и <code>setInterval</code>
для проверки новой почты или твитов. Браузер никак не сможет догадаться. Если вы
каждые несколько секунд проверяете письма, то всё замечательно, но как насчёт
отрисовки 1000 объектов в WebGL? Вы в буквальном смысле будете вести DOS-атаку
на компьютер пользователя через отрисовку в закрытой вкладке объектов, которые
пользователь даже увидеть не может.
</p><p>
<code>requestAnimationFrame</code> решает обе эти проблемы. Функция вызывается в
нужное время, чтобы ваша анимация была синхронизована с экраном, и только тогда,
когда вкладку браузера видно.
</p>
</div>
