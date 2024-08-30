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

# React Server Components以降の実装

コメント可否が選択できる、ブログ記事コンポーネントの例

```tsx {all|6-7|13-18}
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

# React Server Components以降の実装

コメント可否が選択できる、ブログ記事コンポーネントの例

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

## React Server Components<br>≠Server Components

---

# React Server Components=アーキテクチャ

前述の実装例は全てReact Server Componentsの一部である

- Server Components
- Client Components
- Server Actions

![react.devのキャプチャ](/rsc-doc-capture.png)

---
layout: section
---

# React Server Componentsとは<br>何でないのか

---

# Next.js固有のもの？

現状Next.jsのみがサポートしているため、誤解されがち

- React Server ComponentsをNext.jsがサポートしている
- 他のフレームワークもサポートしつつある(対応中のもの含む)
  - [Remix](https://remix.run)
  - [Vike](https://vike.dev)
  - [RedwoodJS](https://redwoodjs.com)

---

# 従来より複雑？

個人的な意見: 総合的に見てシンプル

- Pages Router
  - 共通化しづらい`getServerSideProps`
  - Propsのバケツリレー
  - データ操作のためにtRPCやGraphQLの併用
- App Router
  - コンポーネント指向なデータ取得
  - 組み込みでデータ操作が可能
  - より高いパフォーマンスやシンプルな抽象化

---

# PHP？

[技術の螺旋](https://speakerdeck.com/twada/understanding-the-spiral-of-technologies-2023-edition?slide=10)が見えてれば違うことに気づくはず

- 共通点
  - データフェッチとデータ参照を近くに実装できる(おそらくWordPressなどの実装イメージ？)
- 相違点
  - Reactはコンポーネント指向=CSS・JSをカプセル化可能
  - React Server Componentsでは同一データフェッチをメモ化して重複を排除

<span v-mark="{ at: 1, color: 'red', type: 'underline'}" class="font-bold">共通点を見出し「同じ」と主張してるに過ぎない</span>

---

# islandアーキテクチャ？

TBW: 2層アーキテクチャと言う面では似ている、ただしパフォーマンスやコンポーネント指向データフェッチは実現できてない

---

# GraphQLを置き換えるもの？

TBW: BFF層に絞って考えると、精神的後継と捉えることはできる。ただし、全てのGraphQLを置き換えるわけがない。

memo: https://qiita.com/peka2/items/273be01065a921833878

---
layout: section
---

# React Server Components<br>とは何か(再考)

---
layout: fact
---

## React Server Componentsとは<br>*コンポーネント指向SSRアーキテクチャ*である

---

# _コンポーネント指向SSRアーキテクチャ_

過去の技術の螺旋と比較する

1. リクエスト指向SSR^[従来のWebフレームワークにあったHTML生成はレガシーSSRと考える]: Web MVC
2. コンポーネント指向CSR: React with CSR
3. リクエスト指向SSR+コンポーネント指向CSR: Next.js Pages Router
4. コンポーネント指向SSR: React Server Components

<span v-mark="{ at: 1, color: 'red', type: 'underline'}" class="font-bold">コンポーネント指向なSSRを実現するためのアーキテクチャであることがわかる</span>

---
layout: image

image: /data-fetch-history.png

backgroundSize: contain
---

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
