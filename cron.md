# Cron

#### Create the Magento crontab {#create-the-magento-crontab}

To view the crontab, enter the following command as the Magento file system owner:

```text
crontab -l
```

Version &gt;= 2.2

```text
php bin/magento cron:install [--force]
```

{% hint style="info" %}
magento cron:install does not rewrite an existing crontab inside \#~ MAGENTO START and \#~ MAGENTO END comments in your crontab.

magento cron:install --force has no effect on any cron jobs outside the Magento comments.
{% endhint %}

Version &lt; 2.2 \(manual\)

Terminal edit cron:

```text
crontab -e
```

Add code:

```text
#~ MAGENTO START
* * * * * /usr/bin/php /var/www/html/magento2/bin/magento cron:run 2>&1 | grep -v Ran jobs by schedule >> /var/www/html/magento2/var/log/magento.cron.log
* * * * * /usr/bin/php /var/www/html/magento2/update/cron.php >> /var/www/html/magento2/var/log/update.cron.log
* * * * * /usr/bin/php /var/www/html/magento2/bin/magento setup:cron:run >> /var/www/html/magento2/var/log/setup.cron.log
#~ MAGENTO END
```

#### Remove the Magento crontab: {#config-cron-remove}

```text
php bin/magento cron:remove
```

#### Run cron from the command line:

To run the indexing cron job, enter:

```text
php bin/magento cron:run --group index
```

To run the default cron job, enter:

```text
php bin/magento cron:run --group default
```

{% hint style="info" %}
where --group specifies the cron group to run \(omit this option to run cron for all groups\)
{% endhint %}



