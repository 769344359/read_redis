https://dreampuf.github.io/GraphvizOnline/

> get 命令
```
strict digraph G {

main->aeMain->aeProcessEvents->readQueryFromClient->processInputBuffer->processCommand->call->getCommand->getGenericCommand->lookupKeyReadOrReply->lookupKeyRead->lookupKeyReadWithFlags->lookupKey
main->aeMain->aeProcessEvents->readQueryFromClient->processInputBuffer->processCommand->call->getCommand->getGenericCommand->addReplyBulk->addReplyBulkLen->memcpy_to_buf
main->aeMain->beforeSleep->handleClientsWithPendingWrites->writeToClient->write
}
```
