# web component autofill bugs/inconsisten behaviours

I've been playing with Lit/web components lately to work out how realistic it would be for developers to start replacing framework components with web components.  I've run into some frustrating issues around autofilling that I think we should address.  There are 5 files in this doc.

1. [vanilla.html](vanilla.html) is just basic HTML, no web components, with a `submit` handler to save the `form` data to `localStorage`.
1. [webcomponent.html](webcomponent.html) is the basic web component version
1. [webcomponent-nestedinputs-buttonin.html](webcomponent-nestedinputs-buttonin.html) declares the `form` element outisde the web component, then renders the `label` and `input` elements and `button` inside the web component.
1. [webcomponent-nestedinputs-buttonout.html](webcomponent-nestedinputs-buttonout.html) declares the `form` element outisde the web component, then renders the `label` and `input` elements and `button` **outside** the web component.
1. [webcomponent-slottedinputs.html](webcomponent-slottedinputs.html) adds the username and password inputs via slots.

The code should, in theory, grab whatever the user has input into the four fields and dump them into local storage as JSON when the user clicks the "Save details" `button`.  This should trigger Chrome to remember the username and password for the current domain; I've been running on e.g. localhost:12897, just for uniqueness.  

You would also expect the `firstname` and `lastname` inputs to be populated easily, i.e. click on the first input and click on your name in the popup UI then see the rest of the inputs fill, especially after the launch of [this feature in Chrome 99](https://chromestatus.com/feature/5769419581030400).

However, the examples have varying behaviour.

## vanilla.html

- `vanilla.html` does what you expect it to - autofills all the fields with your firstname/lastname/email on first visit.
- Once you save details, it autofills the username and password on subsequent visits, and autofills the firstname/lastname when you tell it to.

## webcomponent.html

- Works well in terms of filling in firstname/lastname/username with email address only by selecting the first input and picking an option from the dropdown.
- However, the web component is incapable of autofilling the username/password fields on first load.
- if you start typing in your existing username for the domain, Chrome recognises that you have credentials for the domain and shows the key in the address bar; however, it doesn't fill in the password field.
- if you enter new details in the username/password fields, Chrome will show the key icon in the address bar and you can save the details when you click on it.  It doesn't save the details automatically, you have to actively click the key icon and choose to save.  Not sure if this is WAI because there's no navigation.

## webcomponent-nestedinputs-buttonin.html

This example is to illustrate what might happen if you had input fields nested inside web components so you could do custom validation/styling more easily.

- The submit event on the main DOM never triggers, but this might be an issue with my understanding of event bubbling/composed (I don't really understand these well!)
- This format also cannot autofill stored login credentials.
- The only autofill it can manage is one field at a time, it can't e.g. fill last name automatically based on the selection of first name.

## webcomponent-nestedinputs-buttonout.html

Similar to the previous example but with the submit button _outside_ the web component.

- Very weirdly, this version _can_ access stored login credentials!
- However, when you click the button, although the form's `submit` event is fired, it doesn't collect the form input values.  This may well be WAI, but it feels fiddly and breakable to have to write code that looks into the specific shadowRoot. 

## webcomponents-slottedinputs.html

I saw some folks talk about using slotted elements from the main DOM to make autofill for credentials work so I thought I'd try that.

- The good news: the slotted inputs (just username and password) both autofill with saved credentials.
- the awkward news: the code to handle this use case is workable but inelegant.  If we think about scaling this up to a very complex application, you'd have to keep slotting and passing down, slotting and passing down these elements from the main DOM to wherever they end up, which is a very awkward workaround

# In summary

There's a workable but messy solution here.  It would be great to take a look at better handling of inputs within web components both for providing credentials/autofilling, and also for handling forms in JS.