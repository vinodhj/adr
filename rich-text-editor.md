# ADR: Rich Text Editor

**Context:** We need a customisable rich text editor to provide seamless writing and formatting experience to users.

**Problem:**
We need a solution for text editor which can allow basic text formatting, creation of to-do lists etc which are major features of private notepad service.

**Considered Options:**

1. **[QuillJs :](https://quilljs.com/)** QuillJs is a free, open source WYSIWYG editor built for the modern web.
2. **[EditorJs :](https://editorjs.io/)** EditorJs is a free, open source block-style editor like notion.
3. **[TipTap :](https://tiptap.dev/)** Tiptap is a editor suite which has both free and paid versions.

**Discarded Options:**

1. **QuillJs:**

   1. This does not have some formatting options like checklist, which will be a major feature in notepad.

2. **Tiptap**
   1. This is not open source editor
   2. This has free and paid versions, which can cause limitations in free version in future if some features move from free to paid version of application.

**Decision:** We'll use **EditorJs** for the following reasons:

1. This is free and open source javascript library.
2. This can be easily integrated in any javascript application, not limited to any framework.
3. This has support for many useful formatting options like notion such as checklist, code block etc.
4. This can be configured to output the editor data as clean JSON data, which will be very much useful in storing and maintaining formatted contents.

**Consequences:**

1. Team members will need to familiarize themselves with EditorJs.
