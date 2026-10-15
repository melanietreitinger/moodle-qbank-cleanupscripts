# moodle-qbank-cleanupscripts

> [!CAUTION]
> Use these scripts at your own risk. They are provided as-is and without any warranty.

The scripts address the bug described and fixed in https://moodle.atlassian.net/browse/MDL-83541 and are based on the scripts linked in the comments there.

## Files

| File                                          | Description                                                                                                                                                                                                                                                                                                                     |
|-----------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `question/cleanup_duplicate_subquestions.php`   | Adhoc Task from Moodle 5, triggered in `cleanup_delete_duplicates.php`                                                                                                                                                                                                                                                          |
| `question/cleanup_rename_duplicates.php`        | Rename duplicate questions.<br><br>Recommendes options:<br>`--all`: Process all contexts in the system.<br>`--limit=INT`: Limit the number of questions to process (default: 10000000). ⚠️ Should work with the default value.<br>`--context-aware`: Only mark questions as duplicates within the same context (default: true). |
| `question/cleanup_delete_duplicates.php`        | Delete duplicate questions.<br><br>Recommendes options:<br>`--limit=INT`: Limit the number of questions to process (default: 10000000). ⚠️ Set limit to 100.000 or 200.000                                                                                                                                                      |

> [!WARNING]
> In the "rename" and the "delete" script you can define the memory available to the script. Default is `ini_set('memory_limit', '4G');`. Set to a value that fits your system.
> If you have to choose a lower memory value, you might need to choose a lower the value of `--limit` as well.

> [!NOTE]
> `qtype_matrix` needs adjustment for speeding up deletion: https://github.com/ndunand/moodle-qtype_matrix/issues/147

## Cron
Note that in the beginning, there are many duplicates and it might take several runs of the scripts to clean them all.

After that, a nightly run should be sufficient.

The following cron entries are just an example. They are set to run at 1:05 and 2:05 every day. The first job runs the `cleanup_rename_duplicates.php` script with the `--all` option, which processes all contexts in the system. The second job runs the `cleanup_delete_duplicates.php` script with a limit of 200.000 questions to process. The output of both jobs is redirected to log files in the `/opt/moodledata/cleanuplogs` directory, which has to be created first.

```
# Run search for duplicates
5 1 * * *    php /var/www/html/question/cleanup_rename_duplicates.php --all >>/opt/moodledata/cleanuplogs/cleanup_rename_duplicates-`date +"\%Y-\%m-\%d"`.log 2>&1
# Delete duplicates
5 2 * * *    php /var/www/html/question/cleanup_delete_duplicates.php --limit=200000 >>/opt/moodledata/cleanuplogs/cleanup_delete_duplicates-`date +"\%Y-\%m-\%d-\%H-\%M"`.log 2>&1
```