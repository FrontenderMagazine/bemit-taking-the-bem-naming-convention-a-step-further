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
    

Well we’d definitely start with `user`, but anything beyond that would have to
be a guess, educated or otherwise. However, if we rewrote it with BEM:

    <div class="media  user  user--premium">
      <img src="" alt="" class="media__img  user__photo  avatar" />
      <p class="media__body  user__bio">...</p>
    </div>
    

Here we can instantly see that `user`, `user--premium`, `user__photo`, and 
`user__bio` are all related to each other. We can also see that `media`, 
`media__img`, and `media__body` are related, and that `avatar` is just a lone
Block on its own with no Elements or Modifiers.

This level of detail from our classes alone is great! It allows us to make much
safer and more informed decisions about what things do and how we can use, reuse,
change, or remove them.

The one thing missing from BEM is that it only tells use what classes to in
relative terms, as in, how classes are related to*each other*. They don’t
really give us any idea of how things behave, act, or should be implemented in a
global and non-relative sense.

To that end, I decided to extend BEM to become BEMIT. BEMIT doesn’t add any
other types of class—we still only ever have Blocks, Elements, or Modifiers—but 
it does add usage and state information.

## Namespaces {#namespaces}

So as not to repeat myself, it’s probably best you refer to my post from
earlier this year,[More Transparent UI Code with Namespaces][3], in which I
introduce the idea of prefixing every class in a codebase with a certain string 
in order to explain to developers what kind of job it does. This
[Hungarian notation][4]-like naming allows us to ascertain exactly what kind of
job a class might have, how and where we might be able to reuse it (if at all), 
whether or not we can modify it, and much more. The linked article is pretty 
sizable, but will give you a whole load more insight into the technique.

The most common types of namespace are `c-`, for Components, `o-`, for Objects
, `u-`, for Utilities, and `is-/has-` for States (there are plenty more
detailed in the linked post). With this in mind, the above HTML would be 
rewritten as this:

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