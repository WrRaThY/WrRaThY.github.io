---
layout: post
title:  "Why checkbox styling is so hard?"
date:   2018-04-04 20:20:45 +0200
categories: react reactjs checkbox styling font-awesome fa
---

#### I have no bloody idea!
Someone messed up at some point. Or maybe I'm just too stupid to do frontend.  
For sure I'm not a frontend developer. I just happen to get thrown in that direction from time to time.   
And because of that it's even harder...    

So... If you're a frontend dummy, like me, maybe you struggled with something similar and found out that...
There is no good explanation (that I could find) how to style checkboxes. a step-by-step guide.  

Of course there are tons of posts on SO and there are some blog posts, but I just didn't understand them. Probably because other bloggers think that their readers have some 
knowledge and they can use shortcuts here and there. Well... It took me quite a while to find out what is what...

#### Hopefully I'll fill that void.  
## Step-by-step guide to styling checkboxes using Font-Awesome
and react. Because I'm currently learning it.

first of all lets see 
#### what we're working with.  
![defaults](https://image.ibb.co/jv7Vex/default_checkbox.png)

this code is just a default template that is given by jsfiddle. you can find it [here][default-react-fiddle]  
most important snippet is below. for the whole code - visit jsfiddle linked above.
```html
<label>
  <input type="checkbox" disabled readOnly checked={item.done} /> 
  <span className={item.done ? "done" : ""}>{item.text}</span>
</label>
```

#### the guide
1. you will have to find which icon you want. I suggest doing there [here][fa-icons]    
I have chosen those two: [checked][checked] and [unchecked][unchecked]  
2. apparently, what you have to do is to disappear original checkbox, because who need them checkboxes anyway?!  
```css
input[type=checkbox] {
  display: none;
}
```
3. you might have to* move html elements around  
```html
<input id={`checkbox${idx}`} type="checkbox" checked={this.state.items[idx].done === true} />
<label htmlFor={`checkbox${idx}`}>
  <span className={item.done ? "done" : ""}>{item.text}</span>
</label>
```
because...
4. with a css selector that looks like this: `input[type=checkbox]+label` we can then select our checkbox and label.  
so... in the end our CSS has to look like that:

```css
input[type=checkbox]+label:before {
  display: inline-block;
  font-family: FontAwesome;
  content: "\f096"; /* unchecked icon */
  vertical-align: middle;
  letter-spacing: 10px;
}

input[type=checkbox]:checked+label:before {
  content: "\f046"; /* checked icon */
  font-family: FontAwesome;
  vertical-align: middle;
  letter-spacing: 10px;
}
```

so what do we have here?  
`font-family: FontAwesome` - it's a magic string that just has to be there. it works for __version 4.7.0 of FontAwesome__  
for version 5.0 there are some others, but they are not yet available via npm, so I didn't research that.  

`content: "\f096";` - this is just pure magic. You probably have no idea what this is. __I didn't.__  
when you go back to [the fa website][unchecked] you will see that there is a number there (red arrow)   
![red arrow](https://image.ibb.co/cO9yXH/fa_how_to_use.png)  
small tip: bigger and bigger icons (blue arrow) can be obtained by adding a class (`fa-2x`, `fa-3x`, etc) to your div   

so you can select any icon from font awesome by using this __unicode number__.

and this is it. 

### this should make your checkboxes beautiful :)

if you want to play around - the whole code is available [here][fiddle]

I know I don't have a lot of regularity in my posts, but I have another thing I'd like to share with other react-noobs.
it's again something I had to search for for quite a white and has to do with react.
  
#### stay tuned for the next episode :)
		
		
\* probably there is a way to do that with the original HTML, but I'm just too dumb to do that...  
and yes, this span inside a label is useless, but it was in the original example, so who cares?  

[checked]: https://fontawesome.com/v4.7.0/icon/check-square-o/
[unchecked]: https://fontawesome.com/v4.7.0/icon/square-o/
[fa-icons]: https://fontawesome.com/v4.7.0/icons/
[fiddle]: https://jsfiddle.net/WrRaThY/475Lsn6o/
[default-react-fiddle]: https://jsfiddle.net/boilerplate/react-jsx/
