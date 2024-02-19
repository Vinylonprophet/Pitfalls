# Pitfalls-Http

## Request

### Request-Interception

#### Form-Submission

The interception of form submissions can be done in **two ways**, with the most common approach being the use of event listeners. By listening for the 'submit' event and then utilizing the `preventDefault` method, the default form submission behavior is halted. This allows for the execution of custom logic or validation before the actual submission takes place.

##### 1. Listener

```javascript
document.getElementById('form').addEventListener('submit', function(event) {
  event.preventDefault();
});
```

In my work, I encountered a situation where the form is submitted directly in a script, for example, using `form.submit()`. By manipulating the form submission through scripts, it bypasses my interception, and I cannot prevent the request. In such cases, we can employ the second method.



##### 2. HTMLFormElement

We can intercept by modifying the prototype chain of `HTMLFormElement`. In this case, when we call the `submit()` method and trigger the submit event, the interception logic will take effect.

```javascript
var originalSubmit = HTMLFormElement.prototype.submit;

HTMLFormElement.prototype.submit = function() {
    var action = this.getAttribute('action');
    var method = this.getAttribute('method');

    console.log('Form submission intercepted!');
    
    if (method && method.toLowerCase() === 'post') {
        console.log('POST request to:', action);
    } else {
        console.log('Non-POST request. Method:', method, ' Action:', action);
    }

    originalSubmit.call(this);
};
```



**Considerations:**

1. Listeners are used to intercept button clicks.
2. Modifying the prototype chain of `HTMLFormElement` is employed for intercepting the `submit()` event called by scripts.