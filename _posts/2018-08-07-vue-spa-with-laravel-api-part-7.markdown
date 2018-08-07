---
layout: post
title:  "Vue SPA with Laravel API Part 7"
date:   2018-08-07 18:00:00 +0100
categories: laravel vue
description: "Part 7: Creating the remaining Vue components: Board, List, Card, Modal."
---

The complete source code of this series is available [on GitHub][source].

- Part 1: [Setting up a Vue SPA with a Laravel API as the backend using JWT authentication.][part1]
- Part 2: [Creating everything database related: models, migrations, seeders and factories.][part2]
- Part 3: [Setting up API routes and creating API resource controllers.][part3]
- Part 4: [Testing the API with feature tests and sqlite.][part4]
- Part 5: [Setting up Continuous Deployment with TravisCI and Heroku.][part5]
- Part 6: [Customizing the frontend skeleton, design and Dashboard component.][part6]
- Part 7: Creating the remaining Vue components: Board, List, Card, Modal. (You are here)
- Part 8: Setting up Jest and testing all Vue components with it.

(Links will be added when the posts are online)

In this part we will create all components of our Board view. A Board contains multiple Lists and each List contains multiple Cards. Additionally each Card can spawn a Modal with complete information about the card (like it's description).

`resources/assets/js/components/Board.vue`:

{% raw %}
```html
<template>
  <div class="boards">
    <modal 
      v-if="showModal" 
      :card="modalCard" 
      @close-modal="showModal = false"/>
    <ul v-if="board">
      <input 
        v-if="editing" 
        ref="edit" 
        v-model="newName" 
        type="text" 
        class="input" 
        @keyup.enter="updateBoard" 
        @blur="editing = false">
      <span 
        v-else 
        class="board-name"
        @click="editBoard">Board: {{ board.name }}</span>


      <div>
        <draggable 
          v-model="board.board_lists" 
          :options="{group:'lists', ghostClass:'ghost'}" 
          class="dragArea flex-wrapper" 
          @end="updateListOrder">
          <list 
            v-for="list in orderedList" 
            :list="list" 
            :key="list.order + ',' + board.id + ',' + list.id" 
            :id="list.order"
            @delete-list="deleteList" 
            @update-card-order="updateCardOrder"/>
          <div class="list new-list">
            <form @submit.prevent>
              <input 
                v-model="name" 
                type="text" 
                class="input" 
                placeholder="New List name"
                required>
              <button 
                class="button" 
                @click="createNew">Create</button>
            </form>
          </div>
        </draggable>
      </div>
    </ul>

  </div>
</template>
<style scoped>
.board-name {
    font-size: 1.5rem;
    padding-left: 10px;
}

.flex-wrapper {
    display: flex;
    flex-wrap: nowrap;
    overflow-x: auto;
}

.flex-wrapper .list {
    flex: 0 0 auto;
    width: 270px;
    margin: 5px;
}

.dragArea {
    min-height: 15px;
}
</style>
<script>
import axios from 'axios';
import List from './List.vue';
import Modal from './Modal.vue';
import { EventBus } from '../EventBus.js';
import draggable from 'vuedraggable';
import _ from 'lodash';

export default {
    name: 'Board',
    components: {
        List,
        Modal,
        draggable
    },
    data() {
        return {
            board: null,
            name: "",
            showModal: false,
            modalCard: null,
            editing: false,
            newName: "",
        };
    },
    computed: {
        orderedList: function() {
            return _.orderBy(this.board.board_lists, 'order');
        }
    },
    watch: {
        '$route': 'fetchData'
    },
    created() {
        this.fetchData();
    },
    mounted() {
        EventBus.$on('open-modal', card => {
            this.modalCard = card;
            this.showModal = true;
        });
    },
    methods: {
    fetchData() {
        axios
            .get('/boards/' + this.$route.params.id)
            .then(response => {
                this.board = response.data;
            }).catch(() => {});
    },
    createNew() {
      var order = (this.board.board_lists === undefined ? 0 : this.board.board_lists.length);
        axios
            .post('/boardLists', { board_id: this.board.id, name: this.name, order: order })
            .then(response => {
                this.board.board_lists.push(response.data);
            }).catch(() => {});
            this.name = "";
    },
    deleteList: function (boardlist) {
        this.board.board_lists.splice(this.board.board_lists.indexOf(boardlist), 1);
    },
    updateBoard() {
        this.editing = false;
        axios
            .put('/boards/' + this.board.id, { name: this.newName })
            .then((response) => {
                this.board.name = response.data.name;
            }).catch(() => {});
    },
    editBoard() {
        this.editing = true;
        this.newName = this.board.name
        this.$nextTick(() => this.$refs.edit.focus());
    },
    updateListOrder() {
        var i = 0;
        for (let list of this.board.board_lists) {
          list.order = i;
          i++;
        }

        axios
            .patch('/board/updateListOrder', { board: this.board })
            .then(() => {
                //
            }).catch(() => {});
    },
    updateCardOrder() {
        var i = 0;
        for (let list of this.board.board_lists) {
            i = 0;
            for (let c of list.cards) {
                c.order = i;
                i++;
            }
        }

        axios
            .patch('/board/updateOrder', { board: this.board })
            .then(() => {
                //
            }).catch(() => {});
    }
}
}
</script>
```
{% endraw %}

That's quite a lot so let's start with the script part. The first thing we do is declaring which components we use here. These are the List and draggable components. We need draggable here, since we want the Lists to be draggable. In our data object, board will contain the complete information of our board (containing all lists and cards), name will be bound to the input for creating a new list, editing is a flag that tells us if the board name should be displayed as text or as an input field and newName is the newName of the board if we edit it in the input field.

The computed method uses lodash to order the lists by their order attribute, so it will be displayed in the order that is saved in the backend. Watch will refetch the data if the route changes (this means if we change the url in the browsers address bar it will load the new board we chose). In our methods list fetchData, createNew and updateBoard are pretty simple in that they only do crud stuff. You may ask yourself why do we only remove the element in the array in the deleteList method instead of deleting it with the API? This is because the List component is already doing that and then sending an event to which our Board listens to delete the List in it's view. editBoard will set the editing flag to true, fill the now visible input field with the boards name and on the next tick, when the input is visible it will focus the input. We do this with the help of $refs. As you can see we gave an attribute of `ref="edit"` to the input in our template, with this we can easily find the element. We also set a `@blur` handler on the input. If the input loses focus (the user clicks somewhere else) the editing mode will be done and all changes reset. On enter we will send the changes to the API and display the new name. The editing mode starts by clicking on the board name so it's basically live editing the text.

Lastly we have updateListOrder and updateCardOrder. The first will loop through all lists in their order after being dragged & dropped and assign each an order number. It will then use the API to save this order of the lists. The updateCardOrder method does the same, but it loops through each card in each list.

At the top of the template we have the Modal component. We need to have it in this component because it would be inside a draggable component if we nested it deeper. This would mean we could drag&drop the modal.

The list design is quite simple: We use a flex wrapper on the draggable element, and flex items on the list elements. The flex wrapper will not break but overflow in x. This means each List will be placed to the right of the previous and we never start a new line, but make it scrollable to the right. The new List form is another flex item which is always last in the list. We use two options on the draggable component: group, which will keep list and card dragging distinct and ghostclass, which will alter the design of dragging. The chosen ghostClass here will display a preview of how the elements will look before dropping the element. `@end` calls the updateOrder method after dropping an element.

There are a whole bunch of bindings and event handlers in there, but they are mostly pretty straightforward. Those event handlers containing a dash are events emitted by child components at which we will look now starting with `resources/assets/js/components/List.vue`:

{% raw %}
```html
<template>
  <div class="list-wrapper">
    <div class="card list">
      <header class="card-header board-list">
        <p class="card-header-title">
          <input 
            v-if="editing" 
            ref="edit" 
            v-model="newName" 
            type="text" 
            class="input" 
            @keyup.enter="updateList" 
            @blur="editing = false">
          <span v-else>List: {{ list.name }} <span class="list-navs"><a @click.prevent="clickEdit"><i class="fas fa-edit"/></a> <a @click.prevent="deleteThis"><i class="fas fa-trash"/></a></span></span>
        </p>
      </header>
      <div class="card-content">
        <draggable 
          v-model="list.cards" 
          :options="{group:'cards', ghostClass:'ghost'}" 
          class="dragArea" 
          @end="updateOrder">
          <card 
            v-for="card in orderedList" 
            :card="card" 
            :key="card.order + ',' + list.id + ',' + card.id" 
            :id="card.order"
            @delete-card="deleteCard"/>
        </draggable>
      </div>
      <footer class="card-footer">
        <input 
          v-if="showNew" 
          ref="new" 
          v-model="name" 
          type="text" 
          class="input" 
          placeholder="New Card name"
          @keyup.enter="createNew"
          @blur="showNew = false">
        <div v-else><a @click.prevent="clickNew">Create new Card</a></div>
      </footer>
    </div>
  </div>
</template>
<style scoped>
.dragArea {
    min-height: 15px;
}

.list-navs {
  display: none;
  position: absolute;
  right: 12px;
  top: 12px;
}

.board-list:hover .list-navs {
  display: block;
}

.card-footer {
  padding-bottom: 7px;
  padding-top: 7px;
}

.card-footer div {
  margin: auto;
}

.list-wrapper {
    flex: 0 0 auto;
    width: 270px;
    margin: 5px;
}

.list-wrapper .card.list {
    min-width: 270px;
}
</style>
<script>
import axios from 'axios';
import Card from './Card.vue';
import draggable from 'vuedraggable';
import _ from 'lodash';

export default {
    name: 'List',
    components: {
        Card,
        draggable
    },
    props: {
        list: { type: Object, required: true }
    },
    data() {
        return {
            name: "",
            editing: false,
            showNew: false,
            newName: "",
        };
    },
    computed: {
        orderedList: function() {
            return _.orderBy(this.list.cards, 'order');
        }
    },
    methods: {
    createNew() {
        var order = (this.list.cards === undefined ? 0 : this.list.cards.length);
        axios
            .post('/cards', { list_id: this.list.id, name: this.name, order: order })
            .then(response => {
                this.list.cards.push(response.data);
                this.name = "";
                this.showNew = false;
            }).catch(() => {});
    },
    deleteCard: function (card) {
        this.list.cards.splice(this.list.cards.indexOf(card), 1);
    },
    deleteThis() {
        axios
            .delete('/boardLists/' + this.list.id)
            .then(() => {
                this.$emit('delete-list', this.list);
            }).catch(() => {});
    },
    updateList() {
        this.editing = false;
        axios
            .put('/boardLists/' + this.list.id, { name: this.newName })
            .then((response) => {
                this.list.name = response.data.name;
            }).catch(() => {});
    },
    clickEdit() {
        this.editing = true;
        this.newName = this.list.name;
        this.$nextTick(() => this.$refs.edit.focus());
    },
    clickNew() {
        this.showNew = true;
        this.$nextTick(() => this.$refs.new.focus());
    },
    updateOrder() {
        this.$emit('update-card-order', this);
    }
}
}
</script>
```
{% endraw %}

Designwise we use a bulma card, this time however with a header and a footer. The footer contains a link to create a new card, when clicked it will turn into another input that will be focused and will disappear onBlur. The header contains the List name again with a live edit feature, this time however with an edit button instead of clicking the name and a delete button, which will be displayed only on hover. Both methods work the same as we've seen before. We can also see two `this.$emit();` calls which emit the two custom events we listened for in the Board component.

Almost everything is the same as the Board component, which makes sense since a board shows a list of things and a list shows a list of things. So the rest of this component should be pretty simple to understand.

Lets get to the Card component which will be a lot simpler: `resources/assets/js/components/Card.vue`:

{% raw %}
```html
<template>
  <div class="card board-card">
    <div class="card-content">
      <textarea 
        v-if="editing" 
        ref="edit" 
        v-model="newName" 
        type="text" 
        class="textarea" 
        @keyup.enter="updateCard" 
        @blur="editing = false"/>
      <div v-else>
        <span class="card-navs"><a @click.prevent="editCard"><i class="fas fa-edit"/></a> <a @click.prevent="deleteThis"><i class="fas fa-trash"/></a></span>
        <div @click="openModal">{{ card.name }}
        <span v-if="card.description"><br><i class="fas fa-comment"/></span></div>
      </div>
    </div>
  </div>
</template>
<style>
.card-navs {
  display: none;
  position: absolute;
  right: 12px;
}

.board-card {
  background-color: #363636;
  margin-bottom: 5px;
}

.board-card:hover .card-navs {
  display: block;
}

.board-card .card-content {
  padding: 0.75rem;
}
</style>
<script>
import axios from 'axios';
import { EventBus } from '../EventBus.js';

export default {
    name: 'Card',
    props: {
        card: { type: Object, required: true }
    },
    data() {
        return {
            editing: false,
            newName: "",
        };
    },
    methods: {
    deleteThis() {
        axios
            .delete('/cards/' + this.card.id)
            .then(() => {
                this.$emit('delete-card', this.card);
            }).catch(() => {});
    },
    updateCard() {
        this.editing = false;
        axios
            .put('/cards/' + this.card.id, { name: this.newName })
            .then((response) => {
                this.card.name = response.data.name;
            }).catch(() => {});
    },
    editCard() {
        this.editing = true;
        this.newName = this.card.name;
        this.$nextTick(() => this.$refs.edit.focus());
    },
    openModal() {
        EventBus.$emit('open-modal', this.card);
    }
}
}
</script>
```
{% endraw %}

We again have a bulma card which will be stacked one below the other this time and on hover it will display an edit and delete link. It is again live editing by turning the card text into a textarea which onBlur resets again. The textarea saves on enter, so we can't use new lines inside. But we could change that of course and add a save button. There is nothing really special here besides that, since we have seen everything before.

Lastly the `resources/assets/js/components/Modal.vue` component:

{% raw %}
```html
<template>
  <transition name="modal">
    <div class="modal-mask">
      <div 
        class="modal-wrapper" 
        @click.self="closeModal">
        <div class="modal-container">

          <div class="modal-header">
            <a 
              class="modal-close-button" 
              @click="closeModal">
              <i class="fas fa-times fa-lg"/>
            </a>
            <h2>{{ card.name }}</h2>
          </div>

          <div class="modal-body">
            <h3>Description <small><a @click="startEdit">Edit</a></small></h3>
            <span v-if="!editing">{{ card.description || "No description" }}</span>
            <span v-else>
              <textarea 
                v-model="newDesc" 
                class="textarea"/>
              <a 
                class="button is-success" 
                @click="updateCard">Save</a>
              <a @click="stopEdit">
                <i class="fas fa-times fa-lg"/>
              </a>
            </span>
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>
<style>
h2 {
  font-size: 1.75rem;
}

h3 {
  font-size: 1.25rem;
}

.modal-mask {
  position: fixed;
  z-index: 9998;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, .5);
  display: table;
  transition: opacity .5s ease;
}

.modal-wrapper {
  display: table-cell;
  vertical-align: middle;
}

.modal-container {
  width: 600px;
  height: 600px;
  margin: 0px auto;
  padding: 20px 30px;
  background-color: #242424;
  border-radius: 2px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, .33);
  transition: all .3s ease;
}

.modal-body {
  margin: 20px 0;
}

.modal-body textarea {
    margin-bottom: 15px;
}

.modal-body i {
    margin-top: 10px;
    margin-left: 15px;
}

.modal-close-button {
  float: right;
}

.modal-enter {
  opacity: 0;
}

.modal-leave-active {
  opacity: 0;
}

.modal-enter .modal-container,
.modal-leave-active .modal-container {
  -webkit-transform: scale(1.1);
  transform: scale(1.1);
}
</style>
<script>
import axios from 'axios';

export default {
    name: 'Modal',
    props: {
        card: { type: Object, required: true }
    },
    data() {
        return {
            editing: false,
            newDesc: "",
        };
    },
    methods: {
    updateCard() {
        this.editing = false;
        axios
            .put('/cards/' + this.card.id, { description: this.newDesc })
            .then((response) => {
              this.card.description = response.data.description;
            }).catch(() => {});
    },
    closeModal() {
        this.$emit('close-modal');
    },
    startEdit() {
        this.newDesc = this.card.description;
        this.editing = true;
    },
    stopEdit() {
        this.newDesc = "";
        this.editing = false;
    }
}
}
</script>
```
{% endraw %}

Here we first have a mask, which is a simple black overlay with 50% opacity, which will be behind our modal. The modal itself displays the cards name and it's description. The description is editable, this time not with an onBlur effect. We display an icon to abort the editing process. We also have a scale transform animation, the rest is just some nice designing.

We're almost done with our project. The last post will cover how we set up Jest, add it to TravisCI and write tests for all our components.

[source]: https://github.com/ddmler/boards
[part1]: https://ddmler.github.io/laravel/vue/2018/07/13/vue-spa-with-laravel-api-part-1.html
[part2]: https://ddmler.github.io/laravel/vue/2018/07/17/vue-spa-with-laravel-api-part-2.html
[part3]: https://ddmler.github.io/laravel/vue/2018/07/20/vue-spa-with-laravel-api-part-3.html
[part4]: https://ddmler.github.io/laravel/vue/2018/07/24/vue-spa-with-laravel-api-part-4.html
[part5]: https://ddmler.github.io/laravel/vue/2018/07/27/vue-spa-with-laravel-api-part-5.html
[part6]: https://ddmler.github.io/laravel/vue/2018/07/31/vue-spa-with-laravel-api-part-6.html
