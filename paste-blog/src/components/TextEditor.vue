<template>
  <div class="editor-container">
    <button>Bold</button>
    <editor-content :editor="editor" />
    <n-modal v-model:show="showBeforePasteAlert" preset="dialog" title="Paste Formatting Options"
      content="Would you like to remove the format of the copied content?" positive-text="Remove Formatting"
      negative-text="Keep Formatting" @positive-click="removeFormattingHandler"
      @negative-click="preserveFormattingHandler" />
  </div>
</template>

<script setup>
import { useEditor, EditorContent } from '@tiptap/vue-3';
import StarterKit from '@tiptap/starter-kit';
import { ref } from 'vue';
import Color from '@tiptap/extension-color';
import TextStyle from '@tiptap/extension-text-style';
import { NModal } from 'naive-ui';
import { isURL } from 'validator';

const showBeforePasteAlert = ref(false);
const pastedSlice = ref(null);

// This works fine
const preserveFormattingHandler = () => {
  showBeforePasteAlert.value = false;
  const { from, to } = editor.value.state.selection;
  try {
    const formattedSlice = pastedSlice.value.content
      .toJSON()
      .map((sliced) => {
        if (sliced?.content) {
          sliced.content.map((content) => {
            if (
              content.type === 'text' &&
              isURL(content.text)
            ) {
              const marks = [
                {
                  type: 'link',
                  attrs: {
                    href: content.text,
                    target: '_blank',
                  },
                },
              ];
              return (content.marks = marks);
            }
            return content;
          });
        }
        return sliced;
      });
    editor.value
      .chain()
      .focus()
      .insertContentAt({ from, to }, formattedSlice)
      .run();
  } catch (error) {
    editor.value
      .chain()
      .focus()
      .insertContentAt(
        { from, to },
        pastedSlice.value.content.toJSON()
      )
      .run();
  }
};

// This needs to figure our the correct selection range post content insert.
const removeFormattingHandler = () => {
  showBeforePasteAlert.value = false;
  const { from, to } = editor.value.state.selection;
  try {
    const formattedSlice = pastedSlice.value.content
      .toJSON()
      .map((sliced) => {
        if (sliced?.content) {
          sliced.content.map((content) => {
            if (content?.marks) {
              const marks = content.marks.filter(
                (mark) => mark.type !== 'textStyle'
              );
              return (content.marks = marks);
            } else if (
              content.type === 'text' &&
              isURL(content.text)
            ) {
              // This is for URL paste copied from browser url
              const marks = [
                {
                  type: 'link',
                  attrs: {
                    href: content.text,
                    target: '_blank',
                  },
                },
              ];
              return (content.marks = marks);
            }
            return content;
          });
        } else {
          if (sliced?.marks) {
            const marks = sliced.marks.filter(
              (mark) => mark.type !== 'textStyle'
            );
            sliced.marks = marks;
          }
        }
        return sliced;
      });
    editor.value
      .chain()
      .focus()
      .insertContentAt({ from, to }, formattedSlice)
      .setTextSelection({
        from: from,
        to: from + pastedSlice.value.content.size,
      })
      .run();
  } catch (error) {
    editor.value
      .chain()
      .focus()
      .insertContentAt(
        { from, to },
        pastedSlice.value.content.toJSON()
      )
      .setTextSelection({
        from: from,
        to: from + pastedSlice.value.content.size,
      })
      .run();
  }
};

const handleTransFormTexted = function (slice) {
  showBeforePasteAlert.value = true;
  pastedSlice.value = slice;
  return slice;
};

const content = `<p>I'm running Tiptap with Vue.js. 🎉`;

const editor = useEditor({
  content,
  extensions: [StarterKit, Color, TextStyle],
  editorProps: {
    handlePaste: function () {
      return true;
    },
    transformPasted: handleTransFormTexted,
  },
});
</script>
<style>
.editor-container {
  border-radius: 0.5rem;
  border: 1px solid #d0d5dd;
  padding: 16px;
  height: 100%;
  min-height: 150px;
  min-width: 400px;
  width: 840px;
  margin: 1rem 0;
  font-size: 16px;
  font-family: 'verdana', sans-serif;
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
