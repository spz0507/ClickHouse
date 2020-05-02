---
machine_translated: true
machine_translated_rev: d734a8e46ddd7465886ba4133bff743c55190626
toc_priority: 33
toc_title: "\u062A\u0648\u0632\u06CC\u0639 \u0634\u062F\u0647"
---

# توزیع شده {#distributed}

**جداول با موتور توزیع شده هیچ اطلاعاتی را توسط خود ذخیره نمی کنند**, اما اجازه می دهد پردازش پرس و جو توزیع شده بر روی سرورهای متعدد.
خواندن به طور خودکار موازی. در طول خواندن, شاخص جدول بر روی سرور از راه دور استفاده می شود, اگر وجود دارد.

موتور توزیع پارامترها را می پذیرد:

-   نام خوشه در فایل پیکربندی سرور

-   نام یک پایگاه داده از راه دور

-   نام یک میز از راه دور

-   (اختیاری) sharding کلیدی

-   (اختیاری) نام سیاست, استفاده خواهد شد برای ذخیره فایل های موقت برای ارسال کالاهای کابل

    همچنین نگاه کنید به:

    -   `insert_distributed_sync` تنظیم
    -   [ادغام](../mergetree_family/mergetree.md#table_engine-mergetree-multiple-volumes) برای نمونه

مثال:

``` sql
Distributed(logs, default, hits[, sharding_key[, policy_name]])
```

داده ها از تمام سرورها در ‘logs’ خوشه, از پیش فرض.جدول بازدیدها واقع در هر سرور در خوشه.
داده ها نه تنها به عنوان خوانده شده اما تا حدی بر روی سرور از راه دور پردازش (تا حدی که این امکان پذیر است).
مثلا, برای یک پرس و جو با گروه های, داده خواهد شد بر روی سرور از راه دور جمع, و کشورهای متوسط از توابع دانه خواهد شد به سرور درخواست ارسال. سپس داده ها بیشتر جمع خواهد شد.

به جای نام پایگاه داده, شما می توانید یک عبارت ثابت است که یک رشته را برمی گرداند استفاده. در حال بارگذاری

logs – The cluster name in the server's config file.

خوشه ها مانند این تنظیم می شوند:

``` xml
<remote_servers>
    <logs>
        <shard>
            <!-- Optional. Shard weight when writing data. Default: 1. -->
            <weight>1</weight>
            <!-- Optional. Whether to write data to just one of the replicas. Default: false (write data to all replicas). -->
            <internal_replication>false</internal_replication>
            <replica>
                <host>example01-01-1</host>
                <port>9000</port>
            </replica>
            <replica>
                <host>example01-01-2</host>
                <port>9000</port>
            </replica>
        </shard>
        <shard>
            <weight>2</weight>
            <internal_replication>false</internal_replication>
            <replica>
                <host>example01-02-1</host>
                <port>9000</port>
            </replica>
            <replica>
                <host>example01-02-2</host>
                <secure>1</secure>
                <port>9440</port>
            </replica>
        </shard>
    </logs>
</remote_servers>
```

در اینجا یک خوشه با نام تعریف شده است ‘logs’ که متشکل از دو خرده ریز, که هر کدام شامل دو کپی.
خرده ریز به سرور که شامل بخش های مختلف از داده ها مراجعه (به منظور خواندن تمام داده ها, شما باید تمام خرده ریز دسترسی داشته باشید).
کپی در حال تکثیر سرور (به منظور خواندن تمام داده ها, شما می توانید داده ها بر روی هر یک از کپی دسترسی).

نام خوشه باید حاوی نقطه نیست.

پارامترها `host`, `port` و در صورت تمایل `user`, `password`, `secure`, `compression` برای هر سرور مشخص شده است:
- `host` – The address of the remote server. You can use either the domain or the IPv4 or IPv6 address. If you specify the domain, the server makes a DNS request when it starts, and the result is stored as long as the server is running. If the DNS request fails, the server doesn't start. If you change the DNS record, restart the server.
- `port` – The TCP port for messenger activity (‘tcp\_port’ در پیکربندی, معمولا به مجموعه 9000). نه اشتباه آن را با http\_port.
- `user` – Name of the user for connecting to a remote server. Default value: default. This user must have access to connect to the specified server. Access is configured in the users.xml file. For more information, see the section [حقوق دسترسی](../../../operations/access_rights.md).
- `password` – The password for connecting to a remote server (not masked). Default value: empty string.
- `secure` - استفاده از اس اس ال برای اتصال, معمولا شما همچنین باید تعریف `port` = 9440. سرور باید گوش کند <tcp_port_secure>9440</tcp_port_secure> و گواهی صحیح.
- `compression` - استفاده از فشرده سازی داده ها. مقدار پیش فرض: درست.

When specifying replicas, one of the available replicas will be selected for each of the shards when reading. You can configure the algorithm for load balancing (the preference for which replica to access) – see the [\_تبالسازی](../../../operations/settings/settings.md#settings-load_balancing) تنظیمات.
اگر ارتباط با سرور ایجاد نشده است, وجود خواهد داشت تلاش برای ارتباط با یک ایست کوتاه. اگر اتصال شکست خورده, ماکت بعدی انتخاب خواهد شد, و به همین ترتیب برای همه کپی. اگر تلاش اتصال برای تمام کپی شکست خورده, تلاش تکرار خواهد شد به همان شیوه, چندین بار.
این کار به نفع حالت ارتجاعی, اما تحمل گسل کامل را فراهم نمی کند: یک سرور از راه دور ممکن است اتصال قبول, اما ممکن است کار نمی کند, و یا کار ضعیف.

شما می توانید تنها یکی از خرده ریز مشخص (در این مورد, پردازش پرس و جو باید از راه دور به نام, به جای توزیع) و یا تا هر تعداد از خرده ریز. در هر سفال می توانید از یک به هر تعداد از کپی ها مشخص کنید. شما می توانید تعداد مختلف از کپی برای هر سفال مشخص.

شما می توانید به عنوان بسیاری از خوشه های مشخص که شما در پیکربندی می خواهید.

برای مشاهده خوشه های خود استفاده کنید ‘system.clusters’ جدول

موتور توزیع اجازه می دهد تا کار با یک خوشه مانند یک سرور محلی. با این حال, خوشه غیر قابل اجتنابناپذیری است: شما باید پیکربندی خود را در فایل پیکربندی سرور ارسال (حتی بهتر, برای تمام سرورهای خوشه).

The Distributed engine requires writing clusters to the config file. Clusters from the config file are updated on the fly, without restarting the server. If you need to send a query to an unknown set of shards and replicas each time, you don't need to create a Distributed table – use the ‘remote’ تابع جدول به جای. بخش را ببینید [توابع جدول](../../../sql_reference/table_functions/index.md).

دو روش برای نوشتن داده ها به یک خوشه وجود دارد:

اولین, شما می توانید تعریف که سرور به ارسال که داده ها را به و انجام نوشتن به طور مستقیم در هر سفال. به عبارت دیگر, انجام درج در جداول که جدول توزیع “looks at”. این راه حل انعطاف پذیر ترین است که شما می توانید هر طرح شاردینگ استفاده, که می تواند غیر بدیهی با توجه به الزامات منطقه موضوع. این هم بهینه ترین راه حل از داده ها را می توان به خرده ریز های مختلف نوشته شده است به طور کامل به طور مستقل.

دومین, شما می توانید درج در یک جدول توزیع انجام. در این مورد جدول توزیع داده های درج شده در سراسر سرور خود را. به منظور ارسال به یک جدول توزیع, باید یک مجموعه کلید شارژ دارند (پارامتر گذشته). علاوه بر این, اگر تنها یک سفال وجود دارد, عملیات نوشتن بدون مشخص کردن کلید شاردینگ کار می کند, چرا که هیچ چیز در این مورد معنی نیست.

هر سفال می تواند وزن تعریف شده در فایل پیکربندی داشته باشد. به طور پیش فرض, وزن به یک برابر است. داده ها در سراسر خرده ریز در مقدار متناسب با وزن سفال توزیع. مثلا, اگر دو خرده ریز وجود دارد و برای اولین بار دارای وزن 9 در حالی که دوم دارای وزن 10, برای اولین بار ارسال خواهد شد 9 / 19 بخش هایی از ردیف, و دوم ارسال خواهد شد 10 / 19.

هر سفال می تواند داشته باشد ‘internal\_replication’ پارامتر تعریف شده در فایل پیکربندی.

اگر این پارامتر قرار است به ‘true’ عملیات نوشتن اولین ماکت سالم را انتخاب می کند و داده ها را می نویسد. با استفاده از این جایگزین اگر جدول توزیع شده “looks at” جداول تکرار. به عبارت دیگر اگر جدول ای که داده ها نوشته می شود خود را تکرار می کند.

اگر قرار است ‘false’ (به طور پیش فرض), داده ها به تمام کپی نوشته شده. در اصل این بدان معنی است که توزیع جدول تکرار داده های خود را. این بدتر از استفاده از جداول تکرار شده است زیرا سازگاری کپی ها بررسی نشده است و در طول زمان حاوی اطلاعات کمی متفاوت خواهد بود.

برای انتخاب سفال که یک ردیف از داده های فرستاده شده به sharding بیان تجزيه و تحليل است و آن باقی مانده است از تقسیم آن با وزن کلی خرده ریز. ردیف به سفال که مربوط به نیمه فاصله از باقی مانده از ارسال ‘prev\_weight’ به ‘prev\_weights + weight’ کجا ‘prev\_weights’ وزن کل خرده ریز با کمترین تعداد است, و ‘weight’ وزن این سفال است. مثلا, اگر دو خرده ریز وجود دارد, و برای اولین بار دارای یک وزن 9 در حالی که دوم دارای وزن 10, ردیف خواهد شد به سفال اول برای باقی مانده از محدوده ارسال \[0, 9), و دوم برای باقی مانده از محدوده \[9, 19).

بیان شاردینگ می تواند هر عبارت از ثابت ها و ستون های جدول که یک عدد صحیح را برمی گرداند. برای مثال شما می توانید با استفاده از بیان ‘rand()’ برای توزیع تصادفی داده ها یا ‘UserID’ برای توزیع توسط باقی مانده از تقسیم شناسه کاربر (سپس داده ها از یک کاربر تنها بر روی یک سفال تنها اقامت, که ساده در حال اجرا در و پیوستن به کاربران). اگر یکی از ستون ها به طور مساوی توزیع نشده باشد می توانید در یک تابع هش قرار دهید: اینتاش64 (شناسه).

یک یادآوری ساده از این بخش محدود است راه حل برای sharding و نیست همیشه مناسب است. این برای حجم متوسط و زیادی از داده ها کار می کند (ده ها تن از سرور), اما نه برای حجم بسیار زیادی از داده ها (صدها سرور یا بیشتر). در مورد دوم با استفاده از sharding طرح های مورد نیاز منطقه موضوع را به جای استفاده از مطالب موجود در توزیع جداول.

SELECT queries are sent to all the shards and work regardless of how data is distributed across the shards (they can be distributed completely randomly). When you add a new shard, you don't have to transfer the old data to it. You can write new data with a heavier weight – the data will be distributed slightly unevenly, but queries will work correctly and efficiently.

شما باید نگران sharding طرح در موارد زیر:

-   نمایش داده شد استفاده می شود که نیاز به پیوستن به داده ها (در یا پیوستن) توسط یک کلید خاص. اگر داده ها توسط این کلید پنهان, شما می توانید محلی در استفاده و یا پیوستن به جای جهانی در یا جهانی ملحق, که بسیار موثر تر است.
-   تعداد زیادی از سرور استفاده شده است (صدها یا بیشتر) با تعداد زیادی از نمایش داده شد کوچک (نمایش داده شد فردی مشتریان - وب سایت, تبلیغ, و یا شرکای). به منظور نمایش داده شد کوچک به کل خوشه تاثیر نمی گذارد, این باعث می شود حس برای قرار دادن داده ها برای یک مشتری در یک سفال تنها. متناوبا, همانطور که ما در یاندکس انجام داده ام.متریکا, شما می توانید راه اندازی دو سطح شاردینگ: تقسیم کل خوشه را به “layers”, جایی که یک لایه ممکن است از تکه های متعدد تشکیل شده است. داده ها برای یک مشتری تنها بر روی یک لایه قرار دارد اما ذرات را می توان به یک لایه در صورت لزوم اضافه کرد و داده ها به طور تصادفی در داخل توزیع می شوند. جداول توزیع شده برای هر لایه ایجاد می شوند و یک جدول توزیع شده مشترک برای نمایش داده شد جهانی ایجاد می شود.

داده ها ناهمگام نوشته شده است. هنگامی که در جدول قرار داده شده, بلوک داده ها فقط به سیستم فایل های محلی نوشته شده. داده ها به سرور از راه دور در پس زمینه در اسرع وقت ارسال می شود. دوره ارسال داده ها توسط مدیریت [در حال بارگذاری](../../../operations/settings/settings.md#distributed_directory_monitor_sleep_time_ms) و [در حال بارگذاری](../../../operations/settings/settings.md#distributed_directory_monitor_max_sleep_time_ms) تنظیمات. این `Distributed` موتور هر فایل می فرستد با داده های درج شده به طور جداگانه, اما شما می توانید دسته ای از ارسال فایل های با فعال [نمایش سایت](../../../operations/settings/settings.md#distributed_directory_monitor_batch_inserts) تنظیمات. این تنظیم را بهبود می بخشد عملکرد خوشه با استفاده بهتر از سرور محلی و منابع شبکه. شما باید بررسی کنید که داده ها با موفقیت با چک کردن لیست فایل ها (داده ها در حال انتظار برای ارسال) در دایرکتوری جدول ارسال می شود: `/var/lib/clickhouse/data/database/table/`.

اگر سرور متوقف به وجود داشته باشد و یا راه اندازی مجدد خشن بود (مثلا, پس از یک شکست دستگاه) پس از قرار دادن به یک جدول توزیع, داده های درج شده ممکن است از دست داده. اگر بخشی از داده های خراب شده در دایرکتوری جدول شناسایی شود به ‘broken’ دایرکتوری فرعی و دیگر استفاده می شود.

پردازش پرس و جو در سراسر تمام کپی در یک سفال واحد موازی است زمانی که گزینه حداکثر\_پرورالهراپیلاس فعال است. برای کسب اطلاعات بیشتر به بخش مراجعه کنید [بیشینه\_راپرال\_راپیکال](../../../operations/settings/settings.md#settings-max_parallel_replicas).

## ستونهای مجازی {#virtual-columns}

-   `_shard_num` — Contains the `shard_num` (از `system.clusters`). نوع: [UInt32](../../../sql_reference/data_types/int_uint.md).

!!! note "یادداشت"
    از [`remote`](../../../sql_reference/table_functions/remote.md)/`cluster` توابع جدول داخلی ایجاد نمونه موقت از همان توزیع موتور, `_shard_num` در دسترس وجود دارد بیش از حد.

**همچنین نگاه کنید**

-   [ستونهای مجازی](index.md#table_engines-virtual_columns)

[مقاله اصلی](https://clickhouse.tech/docs/en/operations/table_engines/distributed/) <!--hide-->