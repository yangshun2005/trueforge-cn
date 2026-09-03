- 持久化状态必须遵循以下模式：
  ```ts
  yield event; // 消费者持久化，可能抛出异常
  mutateMemory(); // 仅在持久化成功后执行
  ```