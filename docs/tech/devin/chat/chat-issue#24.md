kocya
  金曜日 22:12
@Devin

Issue#24を実装してください。
https://github.com/kocya-dev-org/zenn-ranking/issues/24

リアクションする

11 件の返信

Devin
アプリ  金曜日 22:12
Devin is running, you can send messages in this thread! Messages starting with 'aside' will be ignored. (open webapp)
22:13
Planning
Working on a plan. By default, I will wait for your approval on the plan after I share it with you.
Change default behavior
22:13
了解しました。GitHub Issue #24を確認し、実装に取り組みます。まずは内容を確認してから計画を立てます。

Devin
アプリ  金曜日 22:18
Planning
Here's my new plan (details attached below):
Plan Overview:
* S3からデータを読み込み、DynamoDBに保存する機能を`batchHandler.ts`に追加（GetObjectCommand、PutItemCommandの実装、snake_caseからcamelCaseへの変換）
* 既存の`processArticlesForDate`関数を拡張し、S3保存後にDynamoDBへの保存処理を追加`batchHandler.ts:158-172`
* 新機能のテストを`batchHandler.test.ts`に追加（S3読み込み、プロパティ変換、DynamoDB保存のテスト）
* ローカルでlintとテストを実行して検証し、PRを作成（Issue #24の実装として）
Please review the plan and confirm or request changes.
少なく表示する
22:18
Uploaded file
 
​
*Task description*
実装要件：S3に保存された記事集計データを読み込み、いいね順にソートして上位30件を抽出し、プロパティ名をsnake_caseからcamelCaseに変換してDynamoDBに保存する機能を実装します。具体的には：
​
1. S3のバケット`zenn-ranking-data-bucket`から日付形式`yyyy/mm/yyyymmdd.json`のデータを読み込む
2. 記事データをいいね数（liked_count）でソートし、上位30件を抽出
3. プロパティ名をsnake_caseからcamelCaseに変換
4. DynamoDBテーブル`zenn-ranking-analysis-daily-table`に保存（キー: `yyyy-mm-dd`、値: 記事情報のJSON文字列）
​
*Procedure (all steps, including verification strategy, git strategy if any)*
​
1. 新しいブランチを作成
```
cd ~/repos/zenn-ranking
git checkout main
git pull
git checkout -b devin/$(date +%s)-implement-s3-to-dynamodb
```
​
2. DynamoDBクライアントをインストール（すでに完了）
```
cd ~/repos/zenn-ranking/back
npm install @aws-sdk/client-dynamodb
```
​
3. S3からデータを読み込み、処理してDynamoDBに保存する機能の実装
​
`batchHandler.ts`に以下の実装を追加します：
​
```/home/ubuntu/repos/zenn-ranking/back/batch/src/batchHandler.ts:3-4
import { S3Client, PutObjectCommand, GetObjectCommand } from "@aws-sdk/client-s3";
import { DynamoDBClient, PutItemCommand } from "@aws-sdk/client-dynamodb";
```
​
4. DynamoDBクライアントの初期化を追加
​
```/home/ubuntu/repos/zenn-ranking/back/batch/src/batchHandler.ts:17-18
const s3Client = new S3Client({ region: "ap-northeast-1" });
const dynamoDbClient = new DynamoDBClient({ region: "ap-northeast-1" });
```
​
5. RankingArticleインターフェースを追加
​
```/home/ubuntu/repos/zenn-ranking/back/batch/src/batchHandler.ts:37-50
interface RankingArticle {
  id: number;
  title: string;
  commentsCount: number;
  likedCount: number;
  articleType: string;
  publishedAt: string;
  user: {
    id: number;
    username: string;
    name: string;
    avatarSmallUrl: string;
  };
}
```
​
6. snake_caseからcamelCaseへの変換関数の実装
​
```/home/ubuntu/repos/zenn-ranking/back/batch/src/batchHandler.ts:143-180
/**
 * snake_caseの文字列をcamelCaseに変換する関数
 * @param str 変換する文字列
 * @returns 変換後の文字列
 */
export const snakeToCamel = (str: string): string => {
  return str.replace(/_([a-z])/g, (_, char) => char.toUpperCase());
};
​
/**
 * オブジェクトのキーをsnake_caseからcamelCaseに変換する関数
 * @param obj 変換するオブジェクト
 * @returns 変換後のオブジェクト
 */
export const convertKeysToCamelCase = <T>(obj: any): T => {
  if (obj === null || typeof obj !== "object") {
    return obj;
  }
​
  if (Array.isArray(obj)) {
    return obj.map(item => convertKeysToCamelCase(item)) as unknown as T;
  }
​
  const result: any = {};
  for (const key in obj) {
    if (Object.prototype.hasOwnProperty.call(obj, key)) {
      const camelKey = snakeToCamel(key);
      result[camelKey] = convertKeysToCamelCase(obj[key]);
    }
  }
  return result as T;
};
```
​
7. S3からデータを読み込む関数の実装
​
```/home/ubuntu/repos/zenn-ranking/back/batch/src/batchHandler.ts:182-220
/**
 * S3から記事データを読み込む関数
 * @param date 対象日付
 * @returns 記事データの配列
 */
export const readArticlesFromS3 = async (date: dayjs.Dayjs): Promise<Article[]> => {
  try {
    const formattedDate = date.format("YYYY-MM-DD");
    const dateParts = formattedDate.split("-");
    const year = dateParts[0];
    const month = dateParts[1];
    const filename = `${formattedDate.replace(/-/g, "")}.json`;
    const key = `${year}/${month}/${filename}`;
​
    const bucketName = process.env.DATA_BUCKET_NAME;
    if (!bucketName) {
      throw new Error("DATA_BUCKET_NAME environment variable is not set");
    }
​
    const command = new GetObjectCommand({
      Bucket: bucketName,
      Key: key,
    });
​
    const response = await s3Client.send(command);
    const body = await response.Body?.transformToString();
    if (!body) {
      throw new Error("No data returned from S3");
    }
​
    const articles: Article[] = JSON.parse(body);
    return articles;
  } catch (error) {
    console.error(`Error reading articles from S3 for date ${date}:`, error);
    throw error;
  }
};
```
​
8. 記事データをDynamoDBに保存する関数の実装
​
```/home/ubuntu/repos/zenn-ranking/back/batch/src/batchHandler.ts:222-266
/**
 * 記事データをDynamoDBに保存する関数
 * @param articles 保存する記事データの配列
 * @param date 対象日付
 * @returns 保存の成功・失敗
 */
export const saveArticlesToDynamoDB = async (articles: Article[], date: dayjs.Dayjs): Promise<boolean> => {
  try {
    const formattedDate = date.format("YYYY-MM-DD");
    
    // いいね数でソートして上位30件を抽出
    const sortedArticles = [...articles].sort((a, b) => b.liked_count - a.liked_count).slice(0, 30);
    
    // snake_caseからcamelCaseに変換
    const camelCaseArticles: RankingArticle[] = convertKeysToCamelCase<RankingArticle[]>(sortedArticles);
    
    // 保存用のオブジェクト作成
    const contentsObject = {
      articles: camelCaseArticles
    };
    
    // DynamoDBに保存
    const tableName = process.env.DAILY_TABLE_NAME;
    if (!tableName) {
      throw new Error("DAILY_TABLE_NAME environment variable is not set");
    }
    
    const command = new PutItemCommand({
      TableName: tableName,
      Item: {
        "yyyy-mm-dd": { S: formattedDate },
        "contents": { S: JSON.stringify(contentsObject) }
      }
    });
    
    await dynamoDbClient.send(command);
    console.log(`Articles saved to DynamoDB: ${tableName} with key ${formattedDate}`);
    return true;
  } catch (error) {
    console.error(`Error saving articles to DynamoDB for date ${date}:`, error);
    return false;
  }
};
```
​
9. 記事データを処理する関数の修正 - S3保存後にDynamoDB保存処理を追加
​
```/home/ubuntu/repos/zenn-ranking/back/batch/src/batchHandler.ts:158-172
/**
 * 特定の日付の記事を取得してS3に保存する関数
 * @param startDate 対象日付（YYYY-MM-DD形式）
 * @param endDate 対象日付（YYYY-MM-DD形式）
 * @returns 処理の成功・失敗
 */
export const processArticlesForDate = async (startDate: dayjs.Dayjs, endDate: dayjs.Dayjs): Promise<boolean> => {
  try {
    const articles = await fetchArticlesByDate(startDate, endDate);
    console.log(`Fetched ${articles.length} articles for ${startDate} ${endDate}`);
​
    if (articles.length <= 0) {
      console.log(`No articles found for ${startDate} ${endDate}`);
      return true;
    }
    const saveToS3Success = await saveArticlesToS3(articles, endDate);
    if (!saveToS3Success) {
      return false;
    }
    
    // S3に保存したデータを読み込み、DynamoDBに保存
    try {
      const savedArticles = await readArticlesFromS3(endDate);
      const saveToDynamoDBSuccess = await saveArticlesToDynamoDB(savedArticles, endDate);
      return saveToDynamoDBSuccess;
    } catch (error) {
      console.error(`Error processing articles for DynamoDB ${startDate} ${endDate}:`, error);
      return false;
    }
  } catch (error) {
    console.error(`Error processing articles for ${startDate} ${endDate}:`, error);
    return false;
  }
};
```
​
10. テストの追加 - S3からの読み込みとDynamoDBへの保存を検証する単体テストを実装
​
`batchHandler.test.ts`に以下のテストを追加します：
​
```/home/ubuntu/repos/zenn-ranking/back/batch/test/batchHandler.test.ts:9-12
import { S3Client, PutObjectCommand, GetObjectCommand } from "@aws-sdk/client-s3";
import { DynamoDBClient, PutItemCommand } from "@aws-sdk/client-dynamodb";
import { mockClient } from "aws-sdk-client-mock";
import { handler, fetchArticlesByDate, saveArticlesToS3, getStartDayOfPreviousWeek, processArticlesForDate, MAX_API_CALLS, readArticlesFromS3, saveArticlesToDynamoDB, snakeToCamel, convertKeysToCamelCase } from "../src/batchHandler";
```
​
```/home/ubuntu/repos/zenn-ranking/back/batch/test/batchHandler.test.ts:21-22
const s3Mock = mockClient(S3Client);
const dynamoDbMock = mockClient(DynamoDBClient);
```
​
```/home/ubuntu/repos/zenn-ranking/back/batch/test/batchHandler.test.ts:39-44
  beforeEach(() => {
    vi.clearAllMocks();
    mockedAxios.get.mockReset();
    s3Mock.reset();
    dynamoDbMock.reset();
​
    process.env.DATA_BUCKET_NAME = "test-bucket";
    process.env.DAILY_TABLE_NAME = "test-daily-table";
  });
```
​
```/home/ubuntu/repos/zenn-ranking/back/batch/test/batchHandler.test.ts:220-316
  describe("snakeToCamel", () => {
    it("snake_caseをcamelCaseに変換する", () => {
      expect(snakeToCamel("snake_case")).toBe("snakeCase");
      expect(snakeToCamel("multiple_word_string")).toBe("multipleWordString");
      expect(snakeToCamel("already_camelCase")).toBe("alreadyCamelCase");
      expect(snakeToCamel("no_change")).toBe("noChange");
    });
  });
​
  describe("convertKeysToCamelCase", () => {
    it("オブジェクトのキーをsnake_caseからcamelCaseに変換する", () => {
      const input = {
        snake_case: "value",
        nested_object: {
          another_key: "value",
          third_key: "value"
        },
        array_key: [
          { item_key: "value" },
          { another_item_key: "value" }
        ]
      };
​
      const expected = {
        snakeCase: "value",
        nestedObject: {
          anotherKey: "value",
          thirdKey: "value"
        },
        arrayKey: [
          { itemKey: "value" },
          { anotherItemKey: "value" }
        ]
      };
​
      const result = convertKeysToCamelCase(input);
      expect(result).toEqual(expected);
    });
  });
​
  describe("readArticlesFromS3", () => {
    it("S3から記事データを読み込む", async () => {
      const mockArticles = [
        { id: 1, title: "Article 1", published_at: "2025-04-29T12:00:00+09:00", liked_count: 10 },
        { id: 2, title: "Article 2", published_at: "2025-04-29T14:00:00+09:00", liked_count: 5 }
      ];
​
      s3Mock.on(GetObjectCommand).resolves({
        Body: {
          transformToString: () => Promise.resolve(JSON.stringify(mockArticles))
        }
      });
​
      const result = await readArticlesFromS3(dayjs("2025-04-29"));
​
      const calls = s3Mock.commandCalls(GetObjectCommand);
      expect(calls).toHaveLength(1);
​
      const command = calls[0].args[0];
      expect(command.input.Bucket).toBe("test-bucket");
      expect(command.input.Key).toBe("2025/04/20250429.json");
​
      expect(result).toEqual(mockArticles);
    });
​
    it("S3から記事データを読み込む際にエラーが発生した場合はエラーをスローする", async () => {
      s3Mock.on(GetObjectCommand).rejects(new Error("S3 error"));
​
      await expect(readArticlesFromS3(dayjs("2025-04-29"))).rejects.toThrow("S3 error");
    });
  });
​
  describe("saveArticlesToDynamoDB", () => {
    it("記事データをDynamoDBに保存する", async () => {
      dynamoDbMock.on(PutItemCommand).resolves({});
​
      const articles = [
        { id: 1, title: "Article 1", published_at: "2025-04-29T12:00:00+09:00", liked_count: 10, user: { id: 1, username: "user1", name: "User 1", avatar_small_url: "url1" } },
        { id: 2, title: "Article 2", published_at: "2025-04-29T14:00:00+09:00", liked_count: 5, user: { id: 2, username: "user2", name: "User 2", avatar_small_url: "url2" } }
      ];
​
      const result = await saveArticlesToDynamoDB(articles, dayjs("2025-04-29"));
​
      expect(result).toBe(true);
​
      const calls = dynamoDbMock.commandCalls(PutItemCommand);
      expect(calls).toHaveLength(1);
​
      const command = calls[0].args[0];
      expect(command.input.TableName).toBe("test-daily-table");
      expect(command.input.Item["yyyy-mm-dd"].S).toBe("2025-04-29");
      
      // JSONをパースして内容を確認
      const contents = JSON.parse(command.input.Item.contents.S);
      expect(contents.articles).toHaveLength(2);
      expect(contents.articles[0].id).toBe(1);
      expect(contents.articles[0].likedCount).toBe(10);
      expect(contents.articles[0].user.username).toBe("user1");
    });
  });
```
​
11. processArticlesForDateの更新に対するテストを修正
​
```/home/ubuntu/repos/zenn-ranking/back/batch/test/batchHandler.test.ts:181-204
  describe("processArticlesForDate", () => {
    it("記事の取得と保存を正常に処理する", async () => {
      mockedAxios.get.mockResolvedValueOnce({
        data: {
          articles: [
            { id: 1, title: "Article 1", published_at: "2025-04-29T12:00:00+09:00", liked_count: 10 },
            { id: 2, title: "Article 2", published_at: "2025-04-29T14:00:00+09:00", liked_count: 0 ...
折りたたむ
 このスニペットは表示に合わせて切り取られています: フルで見る





kocya
  金曜日 22:22
計画通り進めてください

Devin
アプリ  金曜日 22:22
Planning
計画が承認されました。実装を開始します。

Devin
アプリ  金曜日 22:34
Created PR https://github.com/kocya-dev-org/zenn-ranking/pull/26. I will monitor the PR for results and let you know once it's ready for review.
22:35
PR作成完了
Issue #24の実装が完了し、PRを作成しました：
https://github.com/kocya-dev-org/zenn-ranking/pull/26
実装内容
* S3から記事データを読み込む機能
* いいね数でソートして上位30件を抽出する処理
* snake_caseからcamelCaseへのプロパティ名変換
* DynamoDBへのデータ保存機能
検証結果
すべてのCIチェック（バックエンド、フロントエンド、CDK）が正常に通過しました。
PRの内容をご確認いただき、フィードバックがあればお知らせください。また、Devinセッションの詳細は https://app.devin.ai/sessions/303e4e88d12e40d6b775fe34ca17d75a でご確認いただけます。
22:35
Devin is awaiting your response. Type 'SLEEP' to put Devin to sleep. Devin will auto-sleep in 30 minutes.


kocya
  金曜日 23:50
SLEEP