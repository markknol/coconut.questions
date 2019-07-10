# Coconut.questions

> Questions / answers about [coconut.ui](https://github.com/MVCoconut/).

--- 

#### How do you declare some var inside a render function?

<details><summary><b>Answer</b></summary>

* One option is `function render() { var x = 123; return @hxx'<span>{x}</span>';  }`
* Or you use `<let>`: https://github.com/haxetink/tink_hxx/#let

</details>


#### How to acces children within custom render function

> If you have `<custom>Wow</custom>` which is a render function defined like this `function custom(attr:{})`, 
> is it possible to get/render its children too?

<details><summary><b>Answer</b></summary>

Yep, this is as simple as:
 
```
function custom(attr:{children:coconut.ui.Children}) '<div class="custom">${...attr.children}</div>';
```

* You can pass children as second argument as well
* The rules are a bit complex, to allow for different styles: https://github.com/haxetink/tink_hxx/#tag-semantics

</details>

#### How to update page title with a component

<details><summary><b>Answer</b></summary>

```
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

#### How to create inline scope
 
Is it possible to create/define a new observable scope within the view?  Something like this:

```
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
  
```
class Stateful<T> extends View {
  @:attribute var init:Void->T;
  @:attribute var content:{ final state:T; }->RenderResult;
  var adhocState:T;
  function render() {
    if (adhocState == null) adhocState = init();
    return content({ state: adhocState });
  }
}

<for {i in 0...10}>
  <Stateful init={State.new(false)}>
     <content>
       <button onclick=${state.set(true)}>enable</button>
       <button onclick=${state.set(false)}>disable</button>
     </content>
  </Stateful>
</for>

```
usage
```
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

---
