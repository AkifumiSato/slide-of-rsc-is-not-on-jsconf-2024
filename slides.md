---
theme: default
# https://unsplash.com/ja/%E5%86%99%E7%9C%9F/%E7%B4%AB%E9%BB%84%E8%89%B2%E3%82%AA%E3%83%AC%E3%83%B3%E3%82%B8%E8%89%B2-DKDFBtmZSz8
background: /background.png
highlighter: shiki
lineNumbers: false
title: React Server Componentsとは何であって何でないのか
info: |
  ## React Server Componentsとは何であって何でないのか

class: text-center
drawings:
  persist: false
transition: slide-left
mdc: true
---

# React Server Components再考

React Server Componentsとは何であって何でないのか

---

# Profile

- Name: 佐藤 昭文(X: [akfm_sato](https://x.com/akfm_sato))
  - Next.js
  - Rust
  - テスト設計
  - チーム開発
- Activity
  - https://zenn.dev/akfm
  - [JS Conf 2023](https://main--remarkable-figolla-a694f0.netlify.app/1)
  - [Vercel meetup](https://zesty-basbousa-04576f.netlify.app/1)
  - [Node学園42](https://youtu.be/ONMIjHfitHM?t=9139)
  - [Rust入門本の執筆](https://www.shuwasystem.co.jp/book/9784798067315.html)
  - etc

---

# Agenda

- 今日話すこと
  - React Server Componentsとは何か
  - React Server Componentsとは何でないか
  - React Server Componentsの今後
- 今日はなさないこと
  - React Server Componentsの機能の詳細
  - フレームワーク固有の機能

---
layout: section
---

# React Server Components<br>とは何か

---
transition: fade
---

# React Server Components以降の書き方

著者がコメント可否が選択できる、ブログ記事コンポーネントの例

```tsx {all|5-6|13-14}
// post.tsx
import db from "@/db";
import { CommentEditor } from "comment-editor";

async function Post({ id }: { id: string }) {
  // セキュア・シンプルにサーバーリソースにアクセスできる
  const post = await db.posts.get(id);

  return (
    <article>
      <h1>{post.title}</h1>
      <div>{post.body}</div>
      {post.commentable ? (
        // `<CommentEditor>`は必要な時しかbundleされない
        <CommentEditor postId={id} />
      ) : (
        <p>コメントは許可されていません</p>
      )}
    </article>
  );
}
```

---
transition: fade
---

# React Server Components以降の書き方

著者がコメント可否が選択できる、ブログ記事コンポーネントの例

```tsx {all|2|8}
// comment-editor.tsx
"use client";

import { addComments } from "add-comment";

export function CommentEditor({ postId }: { postId: string }) {
  return (
    <form action={addComments}>
      <input type="hidden" name="postId" value={postId} />
      <input type="text" name="name" />
      <textarea name="comment" />
      <button type="submit">コメントする</button>
    </form>
  );
}
```

---

# React Server Components以降の書き方

著者がコメント可否が選択できる、ブログ記事コンポーネントの例

```ts {all|2|7-11}
// add-comment.ts
"use server";

import db from "@/db";

export async function addComments(formData: FormData) {
  const userName = formData.get("name");
  const postId = formData.get("postId");
  const comment = formData.get("comment");

  await db.comments.add(postId, comment, userName);
}
```

---

# 問題: React Server Componentsとは？

前述の実装例において、React Server Componentsにあたるのはどれでしょう？

- A: `post.tsx`
- B: `comment-editor.tsx`
- C: `add-comment.ts`
- D: すべて

---
layout: fact
---

## 答え: D(すべて)

---
layout: fact
---

## Server Components≠React Server Components

---

# React Server Componentsはアーキテクチャである

前述の実装例は全てReact Server Componentsの一部である

- Server Components
- Client Components
- Server Actions

![react.devのキャプチャ](/rsc-doc-capture.png)

---
layout: section
---

# React Server Components<br>とは何でないのか

---

# Next.js固有のもの？

---

# 従来より複雑なもの？

---

# PHP？

---

# islandアーキテクチャ？

---

# GraphQLを置き換えるもの？

---
layout: section
---

# React Server Components<br>とは何か(2)

---

# 従来からある技術との違い

---
layout: section
---

# React Server Components<br>の今後

---

# React Server Componentsの今後
