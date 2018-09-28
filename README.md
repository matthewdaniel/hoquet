# hoquet
Simple JavaScript templating using Arrays as S-expressions (loosely based on Clojure's Hiccup).

## Todo app example (using the web component mixin)

```javascript
import Hoquet from "./mixin.js";

const states = ["todo", "doing", "done"];

class TodoItem extends Hoquet(HTMLElement) {
  constructor({name, state}) {
    super();
    this.name = name;
    this.render();

    this.shadowSelect(
      ["$li", "li", "querySelector"]
    , ["$x", "x"]
    );

    this.state = state;

    this.$li.addEventListener("click", e => this.toggleState());
    this.$x.addEventListener("click", e => {
      e.stopPropagation();
      this.dispatchEvent(new CustomEvent("item-deleted", {
        composed: true, bubbles: true, detail: {}
      }));
    });
  }

  get state() { return this._state }
  set state(state) {
    this._state = state;
    states.forEach(x => this.$li.classList.remove(x));
    this.$li.classList.add(state);
  }

  toggleState() {
    const currentStateIndex = states.indexOf(this.state);
    this.state = states[
      currentStateIndex >= states.length - 1
        ? 0 : currentStateIndex + 1
    ];
  }

  get template() {
    return (
      ["li"
      , ["span", {class: "name"}, this.name]
      , ["span", {id: "x"}, "x"]
      ]
    );
  }

  get styles() {
    return `
      li {
        cursor: pointer;
        list-style: none;
        padding: 10px 15px;
        background-color: #fff9d4;
        background-color: #fafafa;
        border: 1px solid #ff0;
        margin: 5px 0;
        font-family: sans-serif;
        font-size: 2rem;
      }
      .todo .name { color: #33a; }

      .doing .name { color: orange }
      .doing .name::after { content: " (in progress)" }

      .done .name {
        text-decoration: line-through;
        color: #888;
      }
      .name {
        width: 75%;
      }
      #x {
        text-decoration: none!important;
        display: block;
        float: right;
        background-color: #fff;
        padding: 0 10px;
        font-size: 2rem;
      }
    `;
  }
}

class TodoList extends Hoquet(HTMLElement) {
  constructor() {
    super();
    this.render();

    this.shadowSelect(
      ["$list", "list"],
      ["$input", "new-todo"]
    );

    this.addEventListener("keyup", (e) => {
      switch (e.key) {
      case "Enter":
        this.addItem({name: this.$input.value, state: "todo"});
        this.$input.value = null;
        break;
      }
    });
    this.shadowRoot.addEventListener("item-deleted", (e) => {
      this.$list.removeChild(e.target);
    });
  }

  item(item) {
    return new TodoItem(item);
  }

  addItem(item) {
    this.$list.appendChild(this.item(item));
  }

  get template() {
    return (
      ["div"
      , ["input", {id: "new-todo", type: "text", placeholder: "Enter a todo item"}]
      , ["ul", {id: "list"}, ""]
      ]
    );
  }

  get styles() {
    return `
      #list { margin: 0; padding: 0; }
      #new-todo { font-size: 2rem; margin: 0; padding: 0; width: 100%; }
    `;
  }

}

window.customElements.define("todo-list", TodoList);
window.customElements.define("todo-item", TodoItem);
```
