---
layout: post
title:  "Live editing with Vue.js"
date:   2018-06-22 18:00:00 +0100
categories: vue
description: Making a text live editable by turning it into an input field with Vue and sending the changes to an API.
---

I wanted to create a nice "live editing" feature like Trello has. It works like this: You display the text of your object (for example a user name) and when you click on it or on an edit link nearby, the text changes to an input field (or textarea). Now you can edit the text and on enter it saves the changes to the vue model and the database via an API. If the input loses focus (on blur) it should revert to the original vue model text and discard all changes. This will look something like this (unimportant stuff removed):

```html
<template>

    <input v-if="editing" ref="edit" type="text" class="input" v-model="newName" @keyup.enter="updateList" @blur="editing = false">
    <span v-else>List: {{ list.name }} <a href="#" @click.prevent="clickEdit">(Edit)</a></span>

</template>
<script>

    data() {
        return {
            editing: false,
            newName: "",
        };
    },
    methods: {
    updateList() {
        this.editing = false;
        axios
            .put('/boardLists/' + this.list.id, { name: this.newName })
            .then(response => {
                // success
            }).catch(error => {
                // error
            });
        this.list.name = this.newName;
    },
    clickEdit() {
        this.editing = true;
        this.newName = this.list.name;
        this.$nextTick(() => this.$refs.edit.focus());
    }
    }

</script>
```

I'm not using the list name as a v-model, but instead a new data object of vue. The reason for that is that the changes should be discarded after the blur event. That means if I used the v-model like this I had to reset it, but only if the element loses focus and it wasn't saved. So this way it's simpler.

The same can be done with a textarea, in this case it will save on enter key instead of starting a new line and will disappear on blur. But you could change the enter key binding if you need the new line functionality:

```html
<textarea v-if="editing" type="text" ref="edit" class="textarea" v-model="newName" @keyup.enter="updateCard" @blur="editing = false"></textarea>
<span v-else>{{ card.name }} <a href="#" @click.prevent="editCard">(Edit)</a></span>
```

Lastly you could also make the text itself clickable instead of an edit link:

```html
<span v-else><strong>Card:</strong> <span @click="editing = true">{{ card.name }}</span></span>
```

