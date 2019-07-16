# Coconut.questions

> Questions / answers about [coconut.ui](https://github.com/MVCoconut/).

--- 

## How to set attributes based on a condition

> This doesn't work: `<span class="<if {true}>active</if>">`

<details><summary><b>Answer</b></summary>

For css classes, I suggest
* `<span class={["item" => true, "active" => isActive]}` 
* or `<span class={{ item: true, active: isActive}}`

But better yet,  
```haxe
static var ITEM:tink.domspec.ClassName = "item"; 
static var ACTIVE:tink.domspec.ClassName = "active" 
```
And use it like  `<span class={ITEM.add([ACTIVE => isActive]}`.  
It's using this abstract: <https://github.com/haxetink/tink_domspec/blob/master/src/tink/domspec/ClassName.hx>
 
</details>


## How do you declare some var inside a render function?

<details><summary><b>Answer</b></summary>

* One option is `function render() { var x = 123; return @hxx'<span>{x}</span>';  }`
* Or you use `<let>`: <https://github.com/haxetink/tink_hxx/#let>

</details>

## Why doesn't float translate to a renderable

I get `Float should be coconut.ui.RenderResult`

<details><summary><b>Answer</b></summary>

That's on purpose, because e.g. in Spanish `1.234` means `1234`. In general you don't want `1.40632469e12` to be shown to user, so developers are forced to write their own float to string functions.

</details>


## How to access children within custom render function?

> If you have `<custom>Wow</custom>` which is a render function defined like this `function custom(attr:{})`, 
> is it possible to get/render its children too?

<details><summary><b>Answer</b></summary>

Yep, this is as simple as:
 
```haxe
function custom(attr:{children:coconut.ui.Children}) '<div class="custom">${...attr.children}</div>';
```

* You can pass children as second argument as well
* The rules are a bit complex, to allow for different styles: <https://github.com/haxetink/tink_hxx/#tag-semantics>

</details>

## How to update page title with a component?

<details><summary><b>Answer</b></summary>

```haxe
class PageInfo extends View {
    @:attribute var title:String;
    @:attribute var description:String;

    function render() {
        var document:Document = js.Browser.document;
        document.title = title;
        document.getElementsByTagName("title")[0].innerText = title;
        setMeta("description", description);

        inline function keep(v:String) return Math.random() * v.length > 0 ? "" : "";
        return hxx('<div style="display:none">${keep(title)}${keep(description)}</div>');
    }

    inline function setMeta(tag:String, value:String) {
        document.querySelector('meta[name=\'${tag}\']').setAttribute("content", value);
    }
}
```

This works as simple as `<PageInfo title="${title} - ${pageName}" description="${description}"/>`. It updates nicely when the values change :smiley:

</details>

## How to an approach inline scope inside a view?
 
Is it possible to create/define a new observable scope within the view?  Something like this:

```jsx
<for {i in 0...10}>
 <scope enabled={false}>
    ${Std.string(enabled)}
    <button onclick=${enabled = true}>enable</button>
    <button onclick=${enabled = false}>disable</button>
  </let>
</scope>
```

<details><summary><b>Answer</b></summary>
  
It sorta depends on what you need, that thing will reset when the parent rerenders. You could try something like this:
  
```haxe
class Stateful<T> extends View {
  @:attribute var init:Void->T;
  @:attribute var content:{ final state:T; }->RenderResult;
  var adhocState:T;
  function render() {
    if (adhocState == null) adhocState = init();
    return content({ state: adhocState });
  }
}
```

Then use it like this:

```jsx
<for {i in 0...10}>
  <Stateful init={() -> new SomeModel({ enabled: false })}>
     <content>
       <button onclick=${state.enabled = true}>enable</button>
       <button onclick=${state.enabled = false}>disable</button>
     </content>
  </Stateful>
</for>
```

> Cool, how does `<content>` work here?

Childless views have their child notes interpreted as attributes, a feature I termed [complex attributes](https://github.com/haxetink/tink_hxx/#complex-attributes)

</details>

## Are there coconut.ui components to handle routing ?

<details><summary><b>Answer</b></summary>

1. If you want simple location hash and assuming the state is all in one place, then you could do something like `function viewDidMount() { window.onhashchange = function () {/* set states from hash here*/}; Observable.auto(() -> /* compute url from states*/).bind(url -> window.location.hash = url)}`. `Observable.auto` is what's underpinning every `@:computed` property
and in essence a coconut view is something like `Observable.auto(this.render).bind(applyVDom)`.
1. If you like a more elegant solution, check this out <https://github.com/kevinresol/coconut.router>

</details>

---
