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

# What RSC is(not)?

<p class="text-xl">React Server Componentsとは何であって何でないのか</p>

---

# Profile

- Name: 佐藤 昭文(X: [akfm_sato](https://x.com/akfm_sato))
  - Next.js
  - Rust
  - テスト設計
  - アジャイル
- Activity
  - https://zenn.dev/akfm
    - [Next.jsの考え方](https://zenn.dev/akfm/books/nextjs-basic-principle)
  - [Offers - Next.js v15 アップデート解説イベント](https://offers-jp.connpass.com/event/328878/)
  - [JS Conf 2023](https://main--remarkable-figolla-a694f0.netlify.app/1)
  - [Rust入門本の執筆](https://www.shuwasystem.co.jp/book/9784798067315.html)
  - etc...

---

# Agenda

- 今日話すこと
  - React Server Componentsとは何か
  - React Server Componentsとは何でないか
  - React Server Componentsと従来のアーキテクチャ
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

# React Server Components以降の実装

コメント可否が選択できる、ブログ記事コンポーネントの例

```tsx {all|6-7|13-18}
// post.tsx
import db from "@/db";
import { CommentEditor } from "comment-editor";

async function Post({ id }: { id: string }) {
  // サーバーリソースにアクセスできる
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

# React Server Components以降の実装

コメント可否が選択できる、ブログ記事コンポーネントの例

```tsx {all|2|4,8}
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

# React Server Components以降の実装

コメント可否が選択できる、ブログ記事コンポーネントの例

```ts {all|2|6-12}
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

## Server Components<br>≠<br>React Server Components

---

# React Server Componentsはアーキテクチャ名

前述の実装例は全てReact Server Componentsの一部である

- Server Components
- Client Components
- Server Actions

![react.devのキャプチャ](/rsc-doc-capture.png)

<Arrow x1="360" y1="260" x2="260" y2="260" color="red" />

---
layout: section
---

# React Server Components<br>とは何でないのか

---

# Next.js固有のもの？

現状Next.jsのみがサポートしているため、誤解されがち

- React関連機能
  - Server Components
  - Client Components(`"use client;"`)
  - Server Actions(`"use server;"`)
  - React Cache
  - Streaming SSR
  - etc...
- Next.js固有の機能
  - Cache
  - etc...

---

# 従来より複雑？

筆者の主観だが、正しく比較すればシンプルになっていると思う

| 項目                   | 従来                 | RSC                  |
| ---------------------- | -------------------- | -------------------- |
| データフェッチ         | フレームワーク依存   | 非同期コンポーネント |
| データ操作             | 3rd party library    | Server Actions       |
| バンドルサイズ         | 最適化に限界があった | zero bundle size     |
| コードスプリッティング | 開発者が意識         | 自動で行われる       |

---

# PHP？

[技術の螺旋](https://speakerdeck.com/twada/understanding-the-spiral-of-technologies-2023-edition?slide=10)が見えてれば違うことに気づくはず

- 共通点
  - データフェッチとデータ参照を近くに実装できる(おそらくWordPressなどの実装イメージ？)
- 相違点
  - Reactはコンポーネント指向=CSS・JSをカプセル化可能
  - React Server Componentsでは同一データフェッチをメモ化して重複を排除

---

# islandアーキテクチャ？

これらは従来よりWeb開発でよく採用される、2層アーキテクチャ

- PHP+jQuery
- islandアーキテクチャ
- React Server Components

---

# GraphQLを置き換えるもの？

RSCはGraphQLの精神的後継

- GraphQLもRSCも、FacebookはじめMetaの大規模プロダクトを支える技術基盤として考案された
- 最初のRFCはRelayやGraphQLを先導してきた、[Joe Savona氏](https://x.com/en_js)が書いた

---
layout: section
---

# React Server Components<br>と従来のアーキテクチャ

---

<div class="w-full h-full flex justify-center items-center">
<img src="/legacy.png" alt="Web Backend & Micro Service" class="h-220px">
</div>

---

<div class="w-full h-full flex justify-center items-center">
<img src="/csr.png" alt="React with CSR fetch" class="h-350px">
</div>

---

<div class="w-full h-full flex justify-center items-center">
<img src="/ssr.png" alt="React with SSR" class="h-350px">
</div>

---
transition: fade
---

<div class="w-full h-full flex justify-center items-center">
<img src="/rsc-ssr.png" alt="React Server Components with SSR" class="h-350px">
</div>

---

<div class="w-full h-full flex justify-center items-center">
<img src="/rsc.png" alt="React Server Components" class="h-350px">
</div>

---
layout: section
---

# React Server Components<br>の今後

---

# React Server Componentsの今後

Next.js以外はまだこれから本格的なサポートになる模様

- [Remix](https://remix.run)
- [Vike](https://vike.dev)
- [RedwoodJS](https://redwoodjs.com)

---
layout: section
---

# まとめ

---

# まとめ

React Server Componentsとは

- React Server Componentsはアーキテクチャ名
- Facebookのような巨大プロダクトの基盤技術になりうる技術として考案された
- 従来の技術とは螺旋的違いがあり、サーバー側におけるコンポーネント指向データフェッチを実現するもの
- GraphQLの精神的後継な一面もある

---
layout: section
---

# 付録

---

# Container/Presentational Componentsの再来

TBW

---

# Astro Server Islands

TBW
