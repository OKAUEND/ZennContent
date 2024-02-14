---
title: "StoryBookのAI生成アドオンを検証してみる"
emoji: "🐕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["react","storybook","openai"]
published: false
---

# AI 時代

Github Copilot で開発体験が向上し、なかった時代に戻れない体になってしまいました。
予測候補でどのようなコードが想定されるかを表示してくれるのは、使わなかったとしてもどの方向性に目指すべきか指標とはなります。
その中で、StoryBook 向けの AI 生成のライブラリを見つけたので、試してみようと思います。

# StoryBook Genie

https://github.com/eduardconstantin/storybook-genie

StoryBookGenie は、OpenAI を利用した AI 生成のライブラリです。

```
npx storybook-genie
```

CLI を入力すると、候補のコンポーネントが CLI 上に表示され、選択をすると`StoryBook`のファイルが対象ファイルの階層に生成されます。
使い方としては非常にシンプルですので、使い方の記事ではなく、複数のケースでどのようなファイルが生成されるかを試してみます。

# 検証とは何するの？

フロントエンドでよく使われるパターンをいくつか想定してコンポーネントと Hook を作成してみました。
これらの対象に対して、どのような StoryBook のファイルが生成されるかを試してみます。
また、StoryBook-Genie は OpenAI を利用しているため、使った Token 分だけ費用がかかります。
ですので、対象に対してどれぐらいの Token と費用がかかるかもチェックをしてみたいと思います。

:::message
今回は ChatGPT-3.5 を利用して確認をしています。
:::

### 期待している事

そこで、今回は各ケースを試すときに出来てほしい事を箇条書きにして、それと比較してどれぐらいなのかを確認したいと思います。

※例

- ファイルが生成されている
- 使用 Token 数がどれぐらいでいてほしいか
- StoryBook の使用方法を筆者のレベルより上のを出力できているか

:::message
Token 数ですが、プラットフォーム上で確認する画面を見つけることができなかったのであとで利用金額の合算にしています。
:::

## Case1

構成としては非常にシンプルで、タイトルと 1 行程度の文章があるコンポーネントです。

https://github.com/OKAUEND/storybook-genie/blob/main/src/feature/Case1/CaseOne.tsx

### 確認項目

- 作成される StoryBook の状態を把握

では試してみましょう。

---

```ts:Case
import type {
  Meta,
  StoryObj
} from '@storybook/react';
import {
  Box,
  Typography,
  Stack
} from "@mui/material";
const meta = {
  title: "CaseOne",
  component: CaseOne,
  parameters: {
    layout: 'centered'
  },
  tags: ['autodocs'],
}
as Meta < typeof CaseOne >
  export default meta
type Story = StoryObj < typeof meta >
  export const Default: Story = {
    args: {},
  }
```

### 確認

作成される StoryBook の状態を把握 について
出来ている点

- [x] StoryBook のライブラリのインポートは出来ていますね。
- [x] meta の宣言もできているので、確認はできそう

NG な点

- mui の不要なライブラリもインポートされている
- そもそも、CaseOne のコンポーネントがインポートされていない
- `as Meta`になっており、無理にキャストしているような気がする

## Case2

ケース 1 のコンポーネントよりもテキスト内容が多いコンポーネントです。

https://github.com/OKAUEND/storybook-genie/blob/main/src/feature/Case2/CaseTwo.tsx

### 確認項目

- 長文になったときにどの用に出力されるか

(ちなみに、文章は Google の Gemini で生成したのを使用しています)

では試してみましょう。

---

```ts:Case
import {
  Box,
  Typography,
  Stack,
  List,
  ListItem
} from "@mui/material";
export const CaseTwo = () => {
  return (<>
      <Typography variant="h1" component="h2">
        日本の有名都市の紹介！
      </Typography>
      <List>
        <ListItem>
          <Stack spacing={2}>
            <Typography variant="h2" component="h3">
              東京
            </Typography>
            <Box>
              東京は日本の首都であり、世界で最も人口の多い都市圏の1つです。この都市は、日本の政治、経済、文化の中心地です。東京には、皇居、東京タワー、渋谷交差点など、数多くの有名な観光スポットがあります。
            </Box>
            <List>
              <ListItem>晴れ</ListItem>
              <ListItem>最高気温: 12℃</ListItem>
              <ListItem>最低気温: 2℃</ListItem>
              <ListItem>湿度: 40%</ListItem>
              <ListItem>風: 北西 5m/s</ListItem>
            </List>
          </Stack>
        </ListItem>
        <ListItem>
          <Stack spacing={2}>
            <Typography variant="h2" component="h3">
              名古屋
            </Typography>
            <Box>
              名古屋は日本の愛知県に位置する都市です。日本の主要な製造業の中心地であり、トヨタ自動車や本田技研工業などの自動車メーカーの本拠地です。名古屋城や熱田神宮など、名古屋には多くの歴史的建造物があります。
            </Box>
            <List>
              <ListItem>晴れ</ListItem>
              <ListItem>最高気温: 13℃</ListItem>
              <ListItem>最低気温: 3℃</ListItem>
              <ListItem>湿度: 40%</ListItem>
              <ListItem>風: 北西 5m/s</ListItem>
            </List>
          </Stack>
        </ListItem>
        <ListItem>
          <Stack spacing={2}>
            <Typography variant="h2" component="h3">
              大阪
            </Typography>
            <Box>
              大阪は日本の大阪府に位置する都市です。日本の商業と文化の中心地であり、「天下の台所」と呼ばれることもあります。大阪には、大阪城、道頓堀、ユニバーサル・スタジオ・ジャパンなど、多くの観光スポットがあります。
            </Box>
            <List>
              <ListItem>晴れ</ListItem>
              <ListItem>最高気温: 14℃</ListItem>
              <ListItem>最低気温: 4℃</ListItem>
              <ListItem>湿度: 40%</ListItem>
              <ListItem>風: 北西 5m/s</ListItem>
            </List>
          </Stack>
        </ListItem>
        <ListItem>
          <Stack spacing={2}>
            <Typography variant="h2" component="h3">
              福岡
            </Typography>
            <Box>
              福岡は日本の福岡県に位置する都市です。九州地方の中心地であり、「博多」の名で知られています。福岡には、博多ラーメン、太宰府天満宮、糸島などの観光スポットがあります。
            </Box>
            <List>
              <ListItem>晴れ</ListItem>
              <ListItem>最高気温: 15℃</ListItem>
              <ListItem>最低気温: 5℃</ListItem>
              <ListItem>湿度: 40%</ListItem>
              <ListItem>風: 北西 5m/s</ListItem>
            </List>
          </Stack>
        </ListItem>
      </List>
    </>);
};
import type {
  Meta,
  StoryObj
} from '@storybook/react';
const meta = {
  title: "CaseTwo",
  component: CaseTwo,
  parameters: {
    layout: 'centered'
  },
  tags: ['autodocs'],
}
as Meta < typeof CaseTwo > ;
export default meta;
type Story = StoryObj < typeof meta > ;
export const Default: Story = {
  args: {},
};
```

### 確認

作成される StoryBook の状態を把握 について
出来ている点

- [x] StoryBook のライブラリのインポートは出来ていますね。
- [x] meta の宣言もできているので、確認はできそう

NG な点

- CaseTwo のコンポーネントを StoryBook のファイル内に直接生成している
- 生成に 15 秒ほど時間を要した

## Case3

ケース 2 の内容を親から Props で受け取り表示をする方法にしたコンポーネントです。

https://github.com/OKAUEND/storybook-genie/blob/main/src/feature/Case3/CaseThree.tsx

### 確認項目

- Props は認識しているか
- StoryBook の args にダミーデータを作成するか

では試してみましょう。

---

```ts:Case
import type {
  Meta,
  StoryObj
} from '@storybook/react';
import {
  Box,
  Typography,
  Stack,
  List,
  ListItem
} from "@mui/material";
type City = {
  name: string;
  description: string;
  weather: {
    status: string;
    maxTemperature: number;
    minTemperature: number;
    humidity: number;
    wind: string;
  };
};
interface Props {
  Cities: City[];
}
export const CaseThree = ({
  Cities
}: Props) => {
  return (<>
      <Typography variant="h1" component="h2">
        日本の有名都市の紹介！
      </Typography>
      <List>
        {Cities.map((city) => (
          <ListItem key={city.name}>
            <Stack spacing={2}>
              <Typography variant="h2" component="h3">
                {city.name}
              </Typography>
              <Box>{city.description}</Box>
              <List>
                <ListItem>{city.weather.status}</ListItem>
                <ListItem>最高気温: {city.weather.maxTemperature}℃</ListItem>
                <ListItem>最低気温: {city.weather.minTemperature}℃</ListItem>
                <ListItem>湿度: {city.weather.humidity}%</ListItem>
                <ListItem>風: {city.weather.wind}</ListItem>
              </List>
            </Stack>
          </ListItem>
        ))}
      </List>
    </>);
};
const meta = {
  title: "CaseThree",
  component: CaseThree,
  parameters: {
    layout: 'centered'
  },
  tags: ['autodocs'],
}
as Meta < typeof CaseThree >
  export default meta
type Story = StoryObj < typeof meta >
  export const DefaultCase: Story = {
    args: {
      Cities: [{
        name: "Tokyo",
        description: "The capital city of Japan.",
        weather: {
          status: "Sunny",
          maxTemperature: 30,
          minTemperature: 20,
          humidity: 60,
          wind: "10 km/h"
        }
      }, {
        name: "Osaka",
        description: "A bustling city in the Kansai region.",
        weather: {
          status: "Cloudy",
          maxTemperature: 25,
          minTemperature: 18,
          humidity: 70,
          wind: "5 km/h"
        }
      }, {
        name: "Kyoto",
        description: "Known for its temples and traditional culture.",
        weather: {
          status: "Rainy",
          maxTemperature: 22,
          minTemperature: 15,
          humidity: 80,
          wind: "8 km/h"
        }
      }]
    }
  }
```

### 確認

作成される StoryBook の状態を把握 について
出来ている点

- [x] Props は認識できており、args も生成出来ている
- [x] args にダミーデータも生成出来ている
- [x] ケース 2 よりも生成が早かった(5 秒程度)

NG な点

- 型を StoryBook 内に再定義している
- コンポーネント内容を StoryBook 内に再定義している

## Case4

入力をしボタンをクリックしたら入力内容が画面へリスト表示されるロジックを持つコンポーネントです。
ロジックは Hook に、useState で値を持つ React の基本みたいな構成にしてみました。

https://github.com/OKAUEND/storybook-genie/blob/main/src/feature/Case4/CaseFour.tsx

### 確認項目

- Hook は認識するのか
- Hook を持つコンポーネントはどういうファイルが生成されるのか
- ロジックがあるコンポーネントの場合 Token 数がどれぐらいか

では試してみましょう。

---

```ts:Case
import type {
  Meta,
  StoryObj
} from '@storybook/react';
import {
  TextField,
  Typography,
  List,
  ListItem,
  Button
} from "@mui/material";
import {
  useInput,
  useTextItems
} from "./hooks";
const meta = {
  title: "CaseFour",
  component: CaseFour,
  parameters: {
    layout: 'centered'
  },
  tags: ['autodocs'],
}
satisfies Meta < typeof CaseFour >
  export default meta
type Story = StoryObj < typeof meta >
  export const Default: Story = {
    args: {},
  }
```

### 確認

作成される StoryBook の状態を把握 について
出来ている点

NG な点

- ボタンクリック等の確認は別途テストファイルで行う必要がありそう

## Case5

API 通信が行われるロジックを持つコンポーネントです。
今回の API 通信は TanstackQuery を利用してみました。

https://github.com/OKAUEND/storybook-genie/blob/main/src/feature/Case4/CaseFour.tsx

なお、今回は事前に msw や storybook の msw アドオンを導入してあります。

### 確認項目

- API 通信を MSW で Mock してくれるか
- StoryBook 上に msw の項目を追加してくれるか

では試してみましょう。

---

```ts:Case
import type {
  Meta,
  StoryObj
} from '@storybook/react';
import {
  CaseFive
} from './CaseFive';
const meta = {
  title: 'CaseFive',
  component: CaseFive,
  parameters: {
    layout: 'centered'
  },
  tags: ['autodocs'],
}
as Meta < typeof CaseFive >
  export default meta
type Story = StoryObj < typeof meta >
  export const Default: Story = {
    args: {},
  }
```

### 確認

作成される StoryBook の状態を把握 について
出来ている点

NG な点

- API 通信に関しては考慮されていない
- msw を args 内にしていない

## 振り返り

5 つのケースを試してきましたが、単純なテキストのみなコンポーネントですと期待に近いものが出力されていたと思います。
逆に、ロジックが絡んだり API 通信が行われたりする場合には、StoyBook の meta の宣言で行う設定を考慮してくれないので、
手直しする必要があり、二度手間になるかなと感じました。

今回試した事で使用した Token 数は

- Context Tokens:3022 Tokens
- Generated Tokens:1960 Tokens

となります。

# あとがき

使えるものは出力した感じだとは思いますが、結構手直しが必要であったりするので、そのまま使うのは厳しいと思います。
Copilot なら、割と学習させて使用者のコーディングに近いお作法を出力することが可能なので、
現状ではすべての作業を AI に委託するよりかは、Copilot のように伴走しつつ助けてもらうのがベターなのかなと今回検証した感じました。
