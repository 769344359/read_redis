https://dreampuf.github.io/GraphvizOnline/

> get 命令
```
strict digraph G {

main->aeMain->aeProcessEvents->readQueryFromClient->processInputBuffer->processCommand->call->getCommand->getGenericCommand->lookupKeyReadOrReply->lookupKeyRead->lookupKeyReadWithFlags->lookupKey
main->aeMain->aeProcessEvents->readQueryFromClient->processInputBuffer->processCommand->call->getCommand->getGenericCommand->addReplyBulk->addReplyBulkLen->memcpy_to_buf
main->aeMain->beforeSleep->handleClientsWithPendingWrites->writeToClient->write
}
```

> write 命令
```
strict digraph G {

main->aeMain->aeProcessEvents->readQueryFromClient->processInputBuffer->processCommand->call->setCommand->setGenericCommand->setKey->dbAdd->dictAdd
main->aeMain->beforeSleep->handleClientsWithPendingWrites->writeToClient->write->WRITE_TO_SOCKET
main->aeMain->beforeSleep->flushAppendOnlyFile->aof_background_fsync->bioCreateBackgroundJob
main->aeMain->aeProcessEvents->readQueryFromClient->processInputBuffer->processCommand->call->propagate->feedAppendOnlyFile->WRITE_FILE
clone->start_thread->bioProcessBackgroundJobs->fdatasync
}
```
