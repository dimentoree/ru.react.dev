---
title: useOptimistic
---

<Intro>

`useOptimistic` -- это хук React, который позволяет оптимистично обновлять UI.

```js
  const [optimisticState, addOptimistic] = useOptimistic(state, updateFn);
```

</Intro>

<InlineToc />

---

## Справочник {/*reference*/}

### `useOptimistic(state, updateFn)` {/*use*/}

`useOptimistic` -- это хук React, который позволяет показать другое состояние, пока выполняется какое-то асинхронное действие. Он принимает состояние, которое может отличаться во время выполнения асинхронного действия, например, таким действием может быть сетевой запрос. Вы передаёте функцию, которая принимает текущее состояние и входные параметры для какого-то действия, и возвращает оптимистичное состояние, которое используется пока действие находится в режиме ожидания.

Это состояние называется «оптимистичным», потому что оно обычно используется для немедленного отображения пользователю результата выполнения действия, даже если само действие требует времени для завершения.

```js
import { useOptimistic } from 'react';

function AppContainer() {
  const [optimisticState, addOptimistic] = useOptimistic(
    state,
    // updateFn
    (currentState, optimisticValue) => {
      // объедините и верните новое состояние
      // с оптимистичным значением
    }
  );
}
```

[Посмотрите больше примеров ниже.](#usage)

#### Параметры {/*parameters*/}

* `state`: значение, которое вернётся изначально, и во всех случаях, когда нет действий в состоянии ожидания.
* `updateFn(currentState, optimisticValue)`: функция, которая принимает текущее состояние и оптимистичное значение, переданное в `addOptimistic`, и возвращает результирующее оптимистичное состояние. Функция должна быть чистой. `updateFn` принимает два параметра: `currentState` и `optimisticValue`. Возвращаемое значение будет результатом объединения `currentState` и `optimisticValue`.


#### Возвращаемое значение {/*returns*/}

* `optimisticState`: Результат оптимистичного состояния. Оно равно `state`, пока нет действий в состоянии ожидания, иначе будет равно значению, возвращённому из `updateFn`.
* `addOptimistic`: `addOptimistic` -- это функция-диспетчер, которая вызывается при оптимистичном обновлении. Она принимает только один аргумент `optimisticValue` любого типа и вызовет `updateFn` с параметрами `state` and `optimisticValue`.

---

## Использование {/*usage*/}

### Оптимистичное обновление форм {/*optimistically-updating-with-forms*/}

Хук `useOptimistic` предоставляет возможность для оптимистичного обновления пользовательского интерфейса перед завершением работы в фоновом режиме, например, сетевого запроса. В контексте форм, эта техника помогает сделать приложение более отзывчивым. Когда пользователь отправляет форму, приложение не ждёт ответ сервера и сразу обновляет интерфейс с учётом ожидаемого результата.

Например, когда пользователь вводит сообщение в форму и нажимает кнопку "Отправить", хук `useOptimistic` позволяет сообщению сразу отобразиться в списке с надписью "Отправка...", ещё до того как сообщение фактически будет отправлено на сервер. Этот "оптимистичный" подход создаёт ощущение скорости и отзывчивости. Затем данные формы действительно отправляются на сервер в фоновом режиме. Как только приходит подтверждение сервера о том, что сообщение получено, надпись "Отправка..." удаляется.

<Sandpack>


```js src/App.js
import { useOptimistic, useState, useRef, startTransition } from "react";
import { deliverMessage } from "./actions.js";

function Thread({ messages, sendMessageAction }) {
  const formRef = useRef();
  function formAction(formData) {
    addOptimisticMessage(formData.get("message"));
    formRef.current.reset();
    startTransition(async () => {
      await sendMessageAction(formData);
    });
  }
  const [optimisticMessages, addOptimisticMessage] = useOptimistic(
    messages,
    (state, newMessage) => [
      {
        text: newMessage,
        sending: true
      },
      ...state,
    ]
  );

  return (
    <>
      <form action={formAction} ref={formRef}>
        <input type="text" name="message" placeholder="Привет!" />
        <button type="submit">Отправить</button>
      </form>
      {optimisticMessages.map((message, index) => (
        <div key={index}>
          {message.text}
          {!!message.sending && <small> (Отправка...)</small>}
        </div>
      ))}
    </>
  );
}

export default function App() {
  const [messages, setMessages] = useState([
    { text: "Привет!", sending: false, key: 1 }
  ]);
  async function sendMessageAction(formData) {
    const sentMessage = await deliverMessage(formData.get("message"));
    startTransition(() => {
      setMessages((messages) => [{ text: sentMessage }, ...messages]);
    })
  }
  return <Thread messages={messages} sendMessageAction={sendMessageAction} />;
}
```

```js src/actions.js
export async function deliverMessage(message) {
  await new Promise((res) => setTimeout(res, 1000));
  return message;
}
```


</Sandpack>
