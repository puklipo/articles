# Laravel Taboos

---

Examples of Laravel projects I wouldn't get involved with even if requested.

## Do Not Write jQuery Directly in Views

Loading an old version of jQuery and writing JS code directly in the view. Even when using Laravel, there are surprisingly many cases like this. If you join such a project midway, investigating what is happening where is incredibly difficult. Even if you take it over, fixing this is impossible, so it's unacceptable.

```html
<html>
<head>
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.12.4/jquery.min.js"></script>
</head>
<body>

<button id="btn">Button</button>
<script>
    $(function () {
        $("#btn").click(function () {
            alert("hello");
        });
    });
</script>

</body>
</html>
```

## Do Not Bring Other Framework's Practices into Laravel
I've seen various things, but this pattern is the one where Laravel is most often used incorrectly.

People make huge mistakes because they don't relearn how to use Laravel and instead forcibly apply the practices of other frameworks to Laravel.

Writing jQuery directly in the view, as mentioned above, is ultimately a derivative of this.

## Do Not Use Versions Whose Support Has Ended
It is essential to use supported versions of Laravel, PHP, and everything else. If you inherit a project, the first task is to upgrade the versions.

- Laravel: https://laravelversions.com/
- PHP: https://www.php.net/supported-versions.php
