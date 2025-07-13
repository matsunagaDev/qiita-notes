---
title: React Router テストでパラメータが取れないときの解決法
tags:
  - React
  - react-router
private: false
updated_at: '2025-05-01T23:56:56+09:00'
id: d761adb77ec52d2e4fad
organization_url_name: jisou
slide: false
ignorePublish: false
---
## はじめに
React Router を使用したのコンポーネントのテストで、以下のエラーが発生しました。

```terminal
 ● UserCard › ユーザー名が表示されていること

    URLパラメータのIDが見つかりません

      40 |         try {
      41 |           if (!id) {
    > 42 |             reject(new Error('URLパラメータのIDが見つかりません'));
         |                    ^
      43 |             return;
      44 |           }
      45 |           console.log(`${id}のユーザー情報を取得します`);
```

これは、ユーザー名が表示されていることをテストした際に表示されたエラー内容です。
今回は、コンポーネントが適切にルートを認識できない場合の対処法をまとめます。

## 問題点
最初、レンダリングでは以下のようにテストを書きました

```tsx:Usercard.test.tsx
render(
  <MemoryRouter initialEntries={['/cards/testuser']}>
    <ChakraProvider>
      <UserCard />
    </ChakraProvider>
  </MemoryRouter>
);
```

このコードでテストを実行すると、useParamsがパラメータを取得できず、コンポーネント内で期待するid変数がundefinedになり、テストが失敗してしまいました。

## 解決策
実際のルーティング構造を再現する必要があるため、`Routes` と `Route` を追加しました。

```tsx
render(
<MemoryRouter initialEntries={['/cards/testuser']}>
  <Routes> // ← この部分
    <Route 
      path="/cards/:id"
      element={
        <ChakraProvider>
          <UserCard />
        </ChakraProvider>
      }
    />
  </Routes>
</MemoryRouter>
);
```
この変更を加えたところ、テストが成功するようになりました！

---

<details><summary><font color="skyblue">コード全体を見る</font>
</summary>

```tsx:Usercard.test.tsx
describe('UserCard', () => {
  beforeEach(async () => {
    // テスト用データの準備
    mockGetUserSkillById.mockResolvedValue({
      user_id: 'testuser',
      name: 'テストユーザー',
      description: '<p>これはテスト用の自己紹介です</p>',
      skills: [
        { id: '1', name: 'React' },
        { id: '2', name: 'TypeScript' },
      ],
      github_id: 'https://github.com/testuser',
      qiita_id: 'https://qiita.com/testuser',
      x_id: 'https://x.com/testuser',
      created_at: '2025-01-01 12:00:00',
    });

    render(
      <MemoryRouter initialEntries={['/cards/testuser']}>
        <Routes>
          <Route
            path="/cards/:id"
            element={
              <ChakraProvider>
                <UserCard />
              </ChakraProvider>
            }
          />
        </Routes>
      </MemoryRouter>
    );
  });

  it('ユーザー名が表示されていること', async () => {
    const userName = await screen.findByTestId('user-name');
    expect(userName).toBeInTheDocument();
    expect(userName).toHaveTextContent('テストユーザー');
  });
  
  // 他のテストケース...
});
```
</details>


## さいごに
`React Router` でURLパラメータを使用するコンポーネントをテストする場合、単に `MemoryRouter` を使用するだけでなく、`Routes` と `Route`コンポーネントを追加して実際のルーティング構造を再現することが重要です。
