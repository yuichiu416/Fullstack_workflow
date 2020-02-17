## Set avg image's position
Add an attribute `transform="translate(5,10)"`

## Enable kayboard navigation
1. Add an index to the state
2. Add an event listener
    ex.
    ```jsx
        componentDidMount(){
            document.addEventListener("keydown", (e) => {
                let lastIdx = this.matches().length - 1;
                let index = this.state.index;
                if(e.key === "Enter"){
                    document.getElementById(`match-${index}`).click();
                }
                if(e.key === "ArrowUp"){
                        index--;
                } else if(e.key === "ArrowDown"){
                        index++;
                }
                if(index < 0){
                    index = lastIdx;
                }
                else if(index > lastIdx)
                    index = 0;
                this.setState({index: index})
            });
        }
    ```
3. Add a class `selected` when rendering the results
   ex.
   ```jsx
       let results = this.matches().map((result, i) => (
            <li key={i} className={i == this.state.index ? "selected" : ""} ></li>
        ));
   ```
4. Set up the css file, add a `background-color` to the `selected` class

## Enable mouse hover navigation
Add an event listener
   ex.
   ```jsx
    document.addEventListener("mouseover", (e) => {
        if(e.target.localName === "li"){
            this.setState({ index: parseInt(e.target.firstChild.id.replace("match-", ""))});
        }
    });
    ```

## Center an element vertically
If it doesn't go to the middle, try using another tag to wrap it and set the container to `flex`, with `justify-contnt: center`
ex.
```css
display: flex;
justify-content: center;
```

## Select all elements with the same name
```css
[class*='cell']{
}
```
[More examples](https://www.w3schools.com/cssref/css_selectors.asp)
Can combine with the pseudo class, too
ex.
```css
[class*='cell']:last-child{
}
```

## Add a favicon (tab icon)
Add this line to the `<head>` tab
```html
    <link rel="icon" href="link-to-some-icon">
```