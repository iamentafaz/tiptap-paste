<script setup>
  import { useEditor, EditorContent } from '@tiptap/vue-3';
  import StarterKit from '@tiptap/starter-kit';
  import ModalComponent from './ModalComponent.vue';
  import { ref } from 'vue';
  import Color from '@tiptap/extension-color';
  import Text from '@tiptap/extension-text';
  import TextStyle from '@tiptap/extension-text-style';

  const showBeforePasteAlert = ref(false);
  const pastedSlice = ref(null);

  // This works fine
  const preserveFormattingHandler = () => {
    showBeforePasteAlert.value = false;
    const { from, to } = editor.value.state.selection;
    editor.value
      .chain()
      .focus()
      .insertContentAt(
        { from, to },
        {
          type: 'doc',
          content: pastedSlice.value.content.toJSON(),
        }
      )
      .run();
  };

  // This needs to figure our the correct selection range post content insert.
  const removeFormattingHandler = () => {
    showBeforePasteAlert.value = false;
    const { from, to } = editor.value.state.selection;
    const formattedSlice = pastedSlice.value.content
      .toJSON()
      .map((slicedContent) => {
        slicedContent.content.map((content) => {
          if (content?.marks) {
            const marks = content.marks.filter(
              (mark) => mark.type !== 'textStyle'
            );
            return (content.marks = marks);
          }
          return content;
        });
        return slicedContent;
      });

    editor.value
      .chain()
      .focus()
      .insertContentAt(
        { from, to },
        {
          type: 'doc',
          content: formattedSlice,
        }
      )
      .setTextSelection({
        from: from,
        to: from + pastedSlice.value.content.size,
      })
      .run();
  };

  const handleTransFormTexted = function (slice) {
    showBeforePasteAlert.value = true;
    pastedSlice.value = slice;
    return slice;
  };

  const editor = useEditor({
    content: '<p>I’m running Tiptap with Vue.js. 🎉</p>',
    extensions: [StarterKit, Color, Text, TextStyle],
    editorProps: {
      handlePaste: function () {
        return true;
      },
      transformPasted: handleTransFormTexted,
    },
  });
  console.log(editor, 'editor');
</script>

<template>
  <div class="editor-container">
    <editor-content :editor="editor" />
    <modal-component :show="showBeforePasteAlert">
      <h3>Paste Formatting</h3>
      <p>Choose to keep or remove the formating of copied content</p>
      <div class="action-container">
        <button @click="removeFormattingHandler">Remove Format</button>
        <button @click="preserveFormattingHandler">Keep Format</button>
      </div>
    </modal-component>
  </div>
</template>
<style>
  .editor-container {
    border-radius: 0.5rem;
    border: 1px solid #d0d5dd;
    padding: 16px;
    height: 100%;
    min-width: 400px;
    width: 840px;
    margin: 1rem 0;
  }
  .action-container {
    display: flex;
    justify-content: flex-end;
  }
  .ProseMirror:focus-visible {
    border: none;
    outline: none;
  }
  .ProseMirror p {
    margin: 0;
  }
</style>
