# Node.js 调研报告

## 构建 Node.js

如果构建目录的路径包含空格，则构建可能会失败。

### 构建 Node.js：

```shell
$ ./configure
$ make -j4
```

我们可以使用 *Ninja* 来加速构建。

该 `-j4` 选项将导致 `make` 同时运行 4 个编译作业，这会减少构建时间。

以上要求 *Python* 为支持的 *Python* 版本。

构建后，设置防火墙规则可以避免在运行测试时弹出窗口要求接受传入的网络连接。

## 安装 Node.js

将此版本的 *Node.js* 安装到系统目录中：

```shell
$ [sudo] make install
```

## 运行测试

要验证构建：

```shell
$ make test-only
```

此时，您已准备好更改代码并重新运行测试。

如果您在提交拉取请求之前运行测试，请使用：

```shell
$ make -j4 test
```

make -j4 test 对代码库进行全面检查，包括运行 linter 和文档测试。

要在不运行测试的情况下运行 linter，请使用 `make lint/ vcbuild lint`。它会检查 *JavaScript*、*C++* 和 *Markdown* 文件。

如果您正在更新测试并希望在单个测试文件中运行测试（例如 `test/parallel/test-stream2-transform.js`）：

```shell
$ tools/test.py test/parallel/test-stream2-transform.js
```

您可以通过提供子系统的名称来为给定的子系统执行整套测试：

```shell
$ tools/test.py child-process
```

您还可以在测试套件目录（例如 `test/message`）中执行测试：

```shell
$ tools/test.py test/message
```

如果要检查其他选项，请使用以下 `--help` 选项参考帮助：

```shell
$ tools/test.py --help
```

注意：在 *Windows* 上，您应该使用 `python3` 可执行文件。例如：`python3 tools/test.py test/message`

您通常可以直接使用节点运行测试：

```shell
$ ./node test/parallel/test-stream2-transform.js
```

### 说明：`./node` 指向您本地的 Node.js 构建。

`make -j4` 如果更改 `lib` 或 `src` 目录中的代码，请记住在测试运行之间重新编译。

这些测试尝试检测对 IPv6 的支持，并在适当时排除 IPv6 测试。如果您的主接口有 IPv6 地址，那么您的环回接口也必须启用 “::1”。对于 *Ubuntu* 上的某些默认安装，情况似乎并非如此。要在 *Ubuntu* 的环回接口上启用 “::1”：

```shell
sudo sysctl -w net.ipv6.conf.lo.disable_ipv6=0
```

如果您的 *IDE* 配置存在，您可以使用 `node-code-ide-configs` 来运行/调试测试。

## 运行覆盖

确保您添加或更改的任何代码都包含在测试中是一种很好的做法。您可以通过在启用覆盖的情况下运行测试套件来做到这一点：

```shell
$ ./configure --coverage
$ make coverage
```

将 `coverage/index.html` 针对 JavaScript 覆盖率和 `coverage/cxxcoverage.html` C++ 覆盖率编写一份详细的覆盖率报告。

如果您只想运行 JavaScript 测试，则不需要运行第一个命令 (`./configure --coverage`)。运行 `make coverage-run-js`, 以独立于 C++ 测试套件执行 JavaScript 测试：

```shell
$ make coverage-run-js
```

如果您正在更新测试并希望收集单个测试文件的覆盖率（例如 `test/parallel/test-stream2-transform.js`）：

```shell
$ make coverage-clean
$ NODE_V8_COVERAGE=coverage/tmp tools/test.py test/parallel/test-stream2-transform.js
$ make coverage-report-js
```

您可以通过提供子系统的名称来收集给定子系统的整个测试套件的覆盖率：

```shell
$ make coverage-clean
$ NODE_V8_COVERAGE=coverage/tmp tools/test.py --mode=release child-process
$ make coverage-report-js
```

该 `make coverage` 命令将一些工具下载到项目根目录。生成覆盖率报告后进行清理：

```shell
$ make coverage-clean
```

## test 测试

`node:test` 模块有助于创建以 *TAP* 格式报告结果的 *JavaScript* 测试。 要访问它：

```js
import test from 'node:test';
```

此模块仅在 `node:` 协议下可用。 以下将不起作用：

```js
import test from 'test';
```

通过 `test` 模块创建的测试由单个函数组成，该函数以三种方式之一进行处理：

同步的函数，如果抛出异常则认为失败，否则认为通过。
返回 `Promise` 的函数，如果 `Promise` 拒绝，则认为该函数失败，如果 `Promise` 解决，则认为该函数通过。
接收回调函数的函数。 如果回调接收到任何真值作为其第一个参数，则认为测试失败。 如果非真值作为第一个参数传给回调，则认为测试通过。 如果测试函数接收到回调函数并且还返回 `Promise`，则测试将失败。
以下示例说明了如何使用 `test` 模块编写测试。

```js
test('synchronous passing test', (t) => {
  // 此测试通过了，因为它没有抛出异常。
  assert.strictEqual(1, 1);
});

test('synchronous failing test', (t) => {
  // 此测试失败，因为它抛出了异常。
  assert.strictEqual(1, 2);
});

test('asynchronous passing test', async (t) => {
  // 此测试通过了，
  // 因为异步函数返回的 Promise 没有被拒绝。
  assert.strictEqual(1, 1);
});

test('asynchronous failing test', async (t) => {
  // 此测试失败，
  // 因为异步函数返回的 Promise 被拒绝。
  assert.strictEqual(1, 2);
});

test('failing test using Promises', (t) => {
  // Promise 也可以直接使用。
  return new Promise((resolve, reject) => {
    setImmediate(() => {
      reject(new Error('this will cause the test to fail'));
    });
  });
});

test('callback passing test', (t, done) => {
  // done() 是回调函数。
  // 当 setImmediate() 运行时，它调用 done() 不带参数。
  setImmediate(done);
});

test('callback failing test', (t, done) => {
  // 当 setImmediate() 运行时，
  // 使用 Error 对象调用 done() 并且测试失败。
  setImmediate(() => {
    done(new Error('callback failure'));
  });
});
```

当测试文件执行时，*TAP* 被写入 *Node.js* 进程的标准输出。 此输出可以被任何理解 *TAP* 格式的测试工具解释。 如果任何测试失败，则进程退出代码设置为 `1`。

## 子测试

测试上下文的 `test()` 方法允许创建子测试。 此方法的行为与顶层 `test()` 函数相同。 以下示例演示了如何创建具有两个子测试的顶层测试。

```js
test('top level test', async (t) => {
  await t.test('subtest 1', (t) => {
    assert.strictEqual(1, 1);
  });

  await t.test('subtest 2', (t) => {
    assert.strictEqual(2, 2);
  });
});
```

在本示例中，`await` 用于确保两个子测试均已完成。 这是必要的，因为父测试不会等待子测试完成。 当父测试完成时仍然未完成的任何子测试将被取消并视为失败。 任何子测试失败都会导致父测试失败。

## 跳过测试

通过将 `skip` 选项传给测试，或调用测试上下文的 `skip()` 方法，可以跳过单个测试。 这两个选项都支持包括在 *TAP* 输出中显示的消息，如下例所示。

```js
// 使用了跳过选项，但没有提供任何消息。
test('skip option', { skip: true }, (t) => {
  // 这段代码永远不会被执行。
});

// 使用了跳过选项，并提供了一条消息。
test('skip option with message', { skip: 'this is skipped' }, (t) => {
  // 这段代码永远不会被执行。
});

test('skip() method', (t) => {
  // 如果测试包含额外的逻辑，则务必返回这里。
  t.skip();
});

test('skip() method with message', (t) => {
  // 如果测试包含额外的逻辑，则务必返回这里。
  t.skip('this is skipped');
});
```

## describe/it 语法

运行测试也可以使用 `describe` 来声明套件和 `it` 来声明测试。 套件用于将相关测试组织和分组在一起。 `it` 是 `test` 的别名，除了没有通过测试上下文，因为嵌套是使用套件完成的。

```js
describe('A thing', () => {
  it('should work', () => {
    assert.strictEqual(1, 1);
  });

  it('should be ok', () => {
    assert.strictEqual(2, 2);
  });

  describe('a nested thing', () => {
    it('should work', () => {
      assert.strictEqual(3, 3);
    });
  });
});
```

`describe` 和 `it` 是从 `node:test` 模块导入的。

```js
import { describe, it } from 'node:test';
```

## `only` 测试

如果 *Node.js* 使用 `--test-only` 命令行选项启动，则可以通过将 `only` 选项传给应该运行的测试来跳过除选定子集之外的所有顶层测试。 当运行带有 `only` 选项集的测试时，所有子测试也会运行。 测试上下文的 `runOnly()` 方法可用于在子测试级别实现相同的行为。

```js
// 假设 Node.js 使用 --test-only 命令行选项运行。
// 设置了 'only' 选项，因此运行此测试。
test('this test is run', { only: true }, async (t) => {
  // 在此测试中，默认运行所有子测试。
  await t.test('running subtest');

  // 可以使用 'only' 选项更新测试上下文以运行子测试。
  t.runOnly(true);
  await t.test('this subtest is now skipped');
  await t.test('this subtest is run', { only: true });

  // 切换上下文以执行所有测试。
  t.runOnly(false);
  await t.test('this subtest is now run');

  // 显式地不要运行这些测试。
  await t.test('skipped subtest 3', { only: false });
  await t.test('skipped subtest 4', { skip: true });
});

// 未设置 'only' 选项，因此跳过此测试。
test('this test is not run', () => {
  // 此代码未运行。
  throw new Error('fail');
});
```

## 无关的异步活动

一旦测试函数完成执行，则 *TAP* 结果会尽快输出，同时保持测试的顺序。 但是，测试函数可能会生成比测试本身寿命更长的异步活动。 测试运行器处理此类活动，但不会延迟报告测试结果以适应它。

在下面的示例中，测试完成时仍然有两个 `setImmediate()` 操作未完成。 第一个 `setImmediate()` 尝试创建新的子测试。 因为父测试已经完成并输出结果，新的子测试立即被标记为失败，并在文件的 TAP 输出的顶层报告。

第二个 `setImmediate()` 创建了 `uncaughtException` 事件。 源自已完成测试的 `uncaughtException` 和 `unhandledRejection` 事件由 `test` 模块处理，并在文件的 *TAP* 输出的顶层报告为诊断警告。

```js
test('a test that creates asynchronous activity', (t) => {
  setImmediate(() => {
    t.test('subtest that is created too late', (t) => {
      throw new Error('error1');
    });
  });

  setImmediate(() => {
    throw new Error('error2');
  });

  // 此行之后测试结束。
});
```

## 从命令行运行测试

可以通过传入 `--test` 标志从命令行调用 *Node.js* 测试运行程序：

```shell
node --test
```

默认情况下，*Node.js* 将递归搜索当前目录以查找匹配特定命名约定的 *JavaScript* 源文件。 匹配文件作为测试文件执行。 有关预期测试文件命名约定和行为的更多信息可以在测试运行器执行模型章节中找到。

或者，可以提供一个或多个路径作为 *Node.js* 命令的最终参数，如下所示。

```shell
node --test test1.js test2.mjs custom_test_dir/
```

在本例中，测试运行程序将执行文件 `test1.js` 和 `test2.mjs`。 测试运行器还将递归搜索 `custom_test_dir/` 目录以查找要执行的测试文件。

## 测试运行器执行模型

当搜索要执行的测试文件时，测试运行器的行为如下：

执行用户显式提供的任何文件。
如果用户没有显式地指定任何路径，则递归搜索当前工作目录中指定的文件，如以下步骤所示。
除非用户显式地提供，否则跳过 `node_modules` 目录。
如果遇到名为 `test` 的目录，则测试运行程序将递归搜索所有 `.js`、`.cjs` 和 `.mjs` 文件。 所有这些文件都被视为测试文件，不需要匹配下面详述的特定命名约定。 这是为了适应将所有测试放在单个 `test` 目录中的项目。
在所有其他目录中，匹配以下模式的 `.js`、`.cjs` 和 `.mjs` 文件被视为测试文件：
`^test$` - 基本名称为字符串 `'test'` 的文件。 示例：`test.js`、`test.cjs`、`test.mjs`。
`^test-.+` - 基本名称以字符串 `'test-'` 开头，后跟一个或多个字符的文件。 示例：`test-example.js`、`test-another-example.mjs`。
`.+[\.\-\_]test$` - 基本名称以 `.test`、`-test` 或 `_test` 结尾的文件，前面有一个或多个字符。 示例：`example.test.js`、`example-test.cjs`、`example_test.mjs`。
*Node.js* 理解的其他文件类型，例如 `.node` 和 `.json`，不会由测试运行程序自动执行，但如果在命令行上显式地提供，则支持。
每个匹配的测试文件都在单独的子进程中执行。 如果子进程以退出代码 0 结束，则认为测试通过。 否则，认为测试失败。 测试文件必须是 *Node.js* 可执行文件，但不需要在内部使用 `node:test` 模块。

## 说明

参考网站：http://nodejs.cn/api-v16/test.html

