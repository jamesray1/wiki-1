# Getting Synced

Parity supports state snapshotting that significantly reduces sync-times and database pruning which reduces disk requirements. Both features are enabled by default on most recent Parity releases. Simply get synced by running:

```bash
$ parity
```

### Warp Synchronization

State snapshotting, or warp-sync, allows for an extremely fast "synchronization" that skips almost all of the block processing, simply injecting the appropriate data directly into the database. To use a snapshot sync, you first need to download a snapshot.

When using Parity **1.4** or **1.5**, you can just run with:

```bash
$ parity --warp
```

... to automatically fetch a recent snapshot from the network, restore from it, and continue syncing. When using Parity **1.6** or **1.7**, `--warp` does nothing as it is enabled by default.

Note, at present, snapshotting does not place all of the block or receipt data into the database. This means you will not get information relating to transactions more than a few days old. This is fine for some usages such as mining, but if you have or need access to historical transaction data (e.g. if you have an account that you've been using with Geth and wish to browse sent transactions) then you probably want to sync normally.

To disable snapshotting, run Parity with:

```bash
$ parity --no-warp
```

CLI reference:

```bash
  --warp                         Does nothing; Warp sync is on by default. (default: true)
  --no-warp                      Disable syncing from the snapshot over the network. (default: false)
```

### Database Pruning

Parity's database pruning mode is enabled by default and only maintains a small journal overlay reducing the disk space required by your Parity node significantly.

To disable storage pruning and to keep all state-trie data, run Parity with:

```bash
$ parity --pruning archive
```

Further fine-tuning of the pruning is enable via the `--pruning history` and `--pruning-memory` flags. Full CLI reference:

```bash
  --pruning METHOD               Configure pruning of the state/storage trie. METHOD
                                 may be one of auto, archive, fast:
                                 archive - keep all state trie data. No pruning.
                                 fast - maintain journal overlay. Fast but 50MB used.
                                 auto - use the method most recently synced or
                                 default to fast if none synced (default: auto).
  --pruning-history NUM          Set a minimum number of recent states to keep when pruning
                                 is active. (default: 64).
  --pruning-memory MB            The ideal amount of memory in megabytes to use to store
                                 recent states. As many states as possible will be kept
                                 within this limit, and at least --pruning-history states
                                 will always be kept. (default: 75)
```

### Troubleshooting / manual fixes until there are more automated fixes

#### How to fix slow syncing block-by-block after snapshotting

Currently if Parity is slowly syncing block-by-block after getting snapshots and will take a very long time to catch up to the current state, then until this bug is fixed, you can do the following.

In the terminal, disable Parity (<kbd>CTRL</kbd> + <kbd>C</kbd>). 

Run `ntpq -p` to sync the system time, while you can also check that it is synced at https://time.is/. Then `cd ~/parity/target/release` and run `./parity db kill`. Remove the chains directory. On Linux, run `rm -r ~/.local/share/io.parity.ethereum/chains/ethereum`. Remove the similar directory in the following folders.
* On Windows: `\AppData\Local\Parity\Ethereum\`
* On Mac: `/Users/you/Library/Application Support/io.parity.ethereum`

Then restart Parity (you can just run `parity`, or navigate to the executable file and run it, e.g. `cd ~/parity/target/release` and run `./parity`).

Check to see the number of snapshots, e.g. `2018-03-09 08:57:43 Syncing snapshot 0/1030`. If you have more than 1000 snapshots, then this should be enough to sync quickly. If you have less than 1000 snapshots, halt Parity and restart it. If there are still less than 1000 snapshots, try again with more peers e.g. with `./parity --peers 128` (the default is 25). And if there are still less than 1000 snapshots, try again later.

You can run `ionice -c 3 parity ui --max-peers 50 --cache-size 1024 --log-file parity-194-startingwarp1.log --port 30304`. `ionice - c 3` gives parity an idle priority, meaning that it will not impact system performance. By default, running `parity` will give it first priority to the disk. You could also explore other options with `ionice --help`. `ui` launches the UI in the browser. `--max-peers 50 --cache-size 1024` is useful e.g. for Intel processors. You can change the log file each time that you run this command if you wish. `--port 30304` is helpful to avoid conflicting with other dapps or processes that may use the same port, e.g. Geth and Akasha, where the latter is not able to change the port number.

If it doesn't display `Syncing snapshot ###/###`, e.g. `Syncing snapshot 299/807`, then restart Parity (<kbd>CTRL</kbd> + <kbd>c</kbd> then run the above command again, `ionice -c 3 parity ui --max-peers 50 --cache-size 1024 --log-file parity-194-startingwarp1.log --port 30304`). Also, if warp sync finishes but it hasn't got up to the latest block (instead it says `Syncing #<block number>), you can also restart Parity.

#### How to fix a port that is already in use

If after running Parity you get the error: `Network port V4(0.0.0.0:30303) is already in use, make sure that another instance of an Ethereum client is not running or change the port using the --port option.`, just try incrementing the port by one and keep doing this until it runs, i.e. start with `./parity --port 30304).
