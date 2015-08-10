#BEMIT: Расширяем BEM CSS нотацию на шаг вперёд
5 августа 2015

Любой, кто читает меня или наблюдает за моей работой даже небольшое время знает, что я горячо рекомендую [BEM CSS нотацию][1].
То, о чем я хотел бы рассказать в этом посте, не является альтернативой или вариантом другого синтаксиса БЭМ, а его развитием:
это небольшие дополнения, который поднимают БЭМ **notch**.
Я назвал это расширение синтаксиса *BEMIT*, т.к. оно берет идеи и некоторые паттерны из (так и не опубликованной)[Inverted Triangle CSS architecture]. BEM + ITCSS = BEMIT.

Напомню БЭМ разделяет все классы на 3 группы:
*   **Блок:** — корень блока-компонента
*   **Элемент:** — часть блока
*   **Модификатор:** — вариация или расширение блока

Блок, Элемент, Модификатор: БЭМ. Абсолютно любой класс в проекте вписывается в одну из этих категорий,
поэтому БЭМ так прекрасен своей простотой и понятностью.

Суть БЭМ в том чтобы сделать ваш код прозрачней и понятней. БЭМ показыват разработчикам как классы относятся один к одному,
что особенно полезно в сложных или глубоких частях DOM. Например, если бы я попросил вас удалить все классы относящиеся к юзеру в этом куске кода,
какие бы вы выбросили?

    <div class="media  user  premium">
      <img src="" alt="" class="img  photo  avatar" />
      <p class="body  bio">...</p>
    </div>
    

Наверное вы бы начали с `user`, но обо всем остальное вы должны были бы догадываться, изучать или пытаться разобраться как-то ещё.
Однако, если мы перепишем это с помощью BEM:

    <div class="media  user  user--premium">
      <img src="" alt="" class="media__img  user__photo  avatar" />
      <p class="media__body  user__bio">...</p>
    </div>
    

Тут мы можем сразу увидеть что `user`, `user--premium`, `user__photo` и
`user__bio` взаимосвязанны. Также мы можем увидеть что `media`,
`media__img` и `media__body` связанны, и что `avatar` это просто одинокий Блок без своих собственных Элементов или Модификаторов.


Круто что мы можем узнать все это только лишь из имен наших классов!
Это позволяет нам принимать более правильные и обдуманные решения про то как работает наш код и как мы можем это использовать, изменять и удалять.

The one thing missing from BEM is that it only tells use what classes to in
relative terms, as in, how classes are related to*each other*. They don’t
really give us any idea of how things behave, act, or should be implemented in a
global and non-relative sense.


В связи с этим я решил расширить БЭМ до BEMIT. BEMIT не добавляет никаких других типов классов, у нас по прежнему остаются только Блоки,
Элементы или Модификаторы, но он добавляет информацию об использовании и состоянии.


## Пространства имён

Чтобы мне не повторятся, вам возможно лучше обраться к моей ранней статье в этом году -
[More Transparent UI Code with Namespaces][3], в котором я представил идею добавления префиксов определенного вида для каждого класса с,
который был пояснял разработчикам что именно этот класс делает. Такое именование в стиле [венгерской нотации][4] позволит нам установить
какую задачу должен выполнять каждый класс, как и где мы может его использовать повторно (если можем), можем мы или нет его модифицировать,
и многое другое. Эта статья довольно большая, но она позволит вам лучше понимать эту технику.

Наиболее распространенные пространства имен это:
`c-` — для компонентов,
`o-` — для объектов,
`u-` — для утилит
и `is-/has-` для состояний (они более подробно описаны в связанной статье).

С учетом этого приведенный HTML может быть переписан следующим образом:

    <div class="o-media  c-user  c-user--premium">
      <img src="" alt="" class="o-media__img  c-user__photo  c-avatar" />
      <p class="o-media__body  c-user__bio">...</p>
    </div>
    

From this, I can see that we have a reusable abstraction in
[the Media Object][5] (`o-media*`) and two implementation-specific components
(`c-user*` and `c-avatar`). These classes are all still Blocks, Elements, or
Modifiers: they haven’t added a new classification, but have added another layer
of rich meaning.

These namespaces tie in with the layers found in the Inverted Triangle CSS
architecture, meaning every class has a specific place to live in our project (
and on our filesystem
).

## Responsive Suffixes {#responsive-suffixes}

The next thing BEMIT adds to traditional BEM naming is responsive suffixes.
These suffixes take the format`@<breakpoint>`, and tell us this class
when at this size:

    <div class="o-media@md  c-user  c-user--premium">
      <img src="" alt="" class="o-media__img@md  c-user__photo  c-avatar" />
      <p class="o-media__body@md  c-user__bio">...</p>
    </div>
    

So here we have `o-media@md`, which means be the media object at this
breakpoint. Other possible examples:

*   `u-hidden@print` – a utility class to hide things when in print context.
*   `u-1/4@lg` – a utility to make something a quarter width in the large
    breakpoint.
   
*   `o-layout@md` – a layout object in the medium breakpoint.

The `@` is a human readable and logical way of denoting conditional states. It
allows developers to learn about any potential permutations or appearances that
the piece of UI in question might have, just at a glance.

**N.B.**: You have to escape the `@` symbol in your CSS file, like so:

    @media print {
    
      .u-hidden\@print {
        display: none;
      }
    
    }
    

## Healthchecks {#healthchecks}

By having these strict and consistent patterns in our HTML we can do a number
of things. Firstly, and most obviously, we can write much more expressive and
rich UI code for our colleagues, meaning they can learn and reason about the
state and shape of a project much more quickly. They can also contribute in the
same consistent manner.

But another happy side effect we get is that we can run a visual healthcheck
against our UI. By using substring selectors, we can target and then visualise
the general makeup of a page based on the types of class it contains:

    /**
     * Outline all classes.
     */
    [class] {
      outline: 5px solid lightgrey;
    }
    
    /**
     * Outline all BEM Elements.
     */
    [class*="__"] {
      outline: 5px solid grey;
    }
    
    /**
     * Outline all BEM Modifiers.
     */
    [class*="--"] {
      outline: 5px solid darkgrey;
    }
    
    /**
     * Outline all Object classes.
     */
    [class^="o-"],
    [class*=" o-"] {
      outline: 5px solid orange;
    }
    
    /**
     * Outline all Component classes.
     */
    [class^="c-"],
    [class*=" c-"] {
      outline: 5px solid cyan;
    }
    
    /**
     * Outline all Responsive classes.
     */
    [class*="@"] {
      outline: 5px solid rosybrown;
    }
    
    /**
     * Outline all Hack classes.
     */
    [class^="_"] {
      outline: 5px solid red;
    }
    

Of course this isn’t bulletproof—something might be both a Component and an
Element and a Responsive class—but if we write the classes in pretty selective
order (i.e. in the order of least to most important to know about, hence Hacks
coming last), we can begin to get a nice visual snapshot of the makeup of any
given page. You can read more about the benefits of this highlighting in
[my previous article about namespaces][6].

We can enable this healthcheck in multiple ways, but the simplest would
probably be nesting the whole lot in[a Scope class][7]:

    .s-healthcheck {
    
      ...
    
      /**
       * Outline all Responsive classes.
       */
      [class*="@"] {
        outline: 5px solid rosybrown;
      }
    
      ...
    
    }
    

…which you can then add to your `html` element when you want to turn it on:

    <html class="s-healthcheck">
    

## Final Word {#final-word}

So there we have it, a couple of simple additions to BEM to turn it into BEMIT
: adding information onto the beginning and end of the standard Block, Element,
or Modifier classes to give us more knowledge about how the classes behave in a
non-relative sense. Some more examples:

    .c-page-head {}
    
    @media screen and (min-width: 15em) {
      .u-text-center\@sm {}
    }
    
    .o-layout__item {}
    
    @media print {
      .u-color-black\@print {}
    }


 [1]: http://csswizardry.com/2013/01/mindbemding-getting-your-head-round-bem-syntax/
 [2]: https://twitter.com/itcss_io
 [3]: http://csswizardry.com/2015/03/more-transparent-ui-code-with-namespaces/
 [4]: https://en.wikipedia.org/wiki/Hungarian_notation

 [5]: http://www.stubbornella.org/content/2010/06/25/the-media-object-saves-hundreds-of-lines-of-code/

 [6]: http://csswizardry.com/2015/03/more-transparent-ui-code-with-namespaces/#highlight-types-of-namespace

 [7]: http://csswizardry.com/2015/03/more-transparent-ui-code-with-namespaces/#scope-namespaces-s-
