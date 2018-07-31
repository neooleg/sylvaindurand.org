---
title: Search engine for<br/> phpBB with <em>Sphinx</em>
---

The phpBB forum engine has an efficient default indexing system, which allows users to search on a large number of messages. However, when a forum exceed millions of messages, it becomes much more efficient and quick to set up an search engine such as *Sphinx*.

## Installation

To install *Sphinx*, you just need to ask:

    sudo apt-get install sphinxsearch

We start by creating it create a user folder:

    sudo mkdir -p /home/sphinx/log

## phpBB-side configuration

In phpBB Admin Control Panel (`ACP` > `General` > `Search settings`), let's choose `Sphinx FullText` with the following parameter:

* Path to data directory: `/home/sphinx/`
* Sphinx search deamon host: `127.0.0.1`
* Sphinx search deamon port: `9312`
* Indexer memory limit: `1024 Mio`

After validation, the Admin Control Panel displays a configuration file that looks like:

    source source_phpbb_[your_id]_main 
    {
        type = mysql # mysql or pgsql 
        sql_host = localhost # SQL server host sphinx connects to 
        sql_user = [dbuser] 
        sql_pass = [dbpassword] 
        sql_db = poudlard 
        sql_port =  # optional, default is 3306 for mysql and 5432 for pgsql 
        sql_query_pre = SET NAMES 'utf8' 
        sql_query_pre = UPDATE phpbb_sphinx SET max_doc_id = (SELECT MAX(post_id) FROM phpbb_posts) WHERE counter_id = 1 
        sql_query_range = SELECT MIN(post_id), MAX(post_id) FROM phpbb_posts 
        sql_range_step = 5000 
        sql_query = SELECT \
                            p.post_id AS id, \
                            p.forum_id, \
                            p.topic_id, \
                            p.poster_id, \
                            p.post_visibility, \
                            CASE WHEN p.post_id = t.topic_first_post_id THEN 1 ELSE 0 END as topic_first_post, \
                            p.post_time, \
                            p.post_subject, \
                            p.post_subject as title, \
                            p.post_text as data, \
                            t.topic_last_post_time, \
                            0 as deleted \
                        FROM phpbb_posts p, phpbb_topics t \
                        WHERE \
                            p.topic_id = t.topic_id \
                            AND p.post_id >= $start AND p.post_id <= $end 
        sql_query_post =  
        sql_query_post_index = UPDATE phpbb_sphinx SET max_doc_id = $maxid WHERE counter_id = 1 
        sql_query_info = SELECT * FROM phpbb_posts WHERE post_id = $id 
        sql_attr_uint = forum_id 
        sql_attr_uint = topic_id 
        sql_attr_uint = poster_id 
        sql_attr_uint = post_visibility 
        sql_attr_bool = topic_first_post 
        sql_attr_bool = deleted 
        sql_attr_timestamp = post_time 
        sql_attr_timestamp = topic_last_post_time 
        sql_attr_string = post_subject 
    }
    source source_phpbb_[your_id]_delta : source_phpbb_[your_id]_main 
    {
        sql_query_pre =  
        sql_query_range =  
        sql_range_step =  
        sql_query = SELECT \
                            p.post_id AS id, \
                            p.forum_id, \
                            p.topic_id, \
                            p.poster_id, \
                            p.post_visibility, \
                            CASE WHEN p.post_id = t.topic_first_post_id THEN 1 ELSE 0 END as topic_first_post, \
                            p.post_time, \
                            p.post_subject, \
                            p.post_subject as title, \
                            p.post_text as data, \
                            t.topic_last_post_time, \
                            0 as deleted \
                        FROM phpbb_posts p, phpbb_topics t \
                        WHERE \
                            p.topic_id = t.topic_id \
                            AND p.post_id >=  ( SELECT max_doc_id FROM phpbb_sphinx WHERE counter_id=1 ) 
    }
    index index_phpbb_[your_id]_main 
    {
        path = /home/sphinx/index_phpbb_[your_id]_main 
        source = source_phpbb_[your_id]_main 
        docinfo = extern 
        morphology = none 
        stopwords =  
        min_word_len = 2 
        charset_type = utf-8 
        charset_table = U+FF10..U+FF19->0..9, 0..9, U+FF41..U+FF5A->a..z, U+FF21..U+FF3A->a..z, A..Z->a..z, a..z, U+0149, U+017F, U+0138, U+00DF, U+00FF, U+00C0..U+00D6->U+00E0..U+00F6, U+00E0..U+00F6, U+00D8..U+00DE->U+00F8..U+00FE, U+00F8..U+00FE, U+0100->U+0101, U+0101, U+0102->U+0103, U+0103, U+0104->U+0105, U+0105, U+0106->U+0107, U+0107, U+0108->U+0109, U+0109, U+010A->U+010B, U+010B, U+010C->U+010D, U+010D, U+010E->U+010F, U+010F, U+0110->U+0111, U+0111, U+0112->U+0113, U+0113, U+0114->U+0115, U+0115, U+0116->U+0117, U+0117, U+0118->U+0119, U+0119, U+011A->U+011B, U+011B, U+011C->U+011D, U+011D, U+011E->U+011F, U+011F, U+0130->U+0131, U+0131, U+0132->U+0133, U+0133, U+0134->U+0135, U+0135, U+0136->U+0137, U+0137, U+0139->U+013A, U+013A, U+013B->U+013C, U+013C, U+013D->U+013E, U+013E, U+013F->U+0140, U+0140, U+0141->U+0142, U+0142, U+0143->U+0144, U+0144, U+0145->U+0146, U+0146, U+0147->U+0148, U+0148, U+014A->U+014B, U+014B, U+014C->U+014D, U+014D, U+014E->U+014F, U+014F, U+0150->U+0151, U+0151, U+0152->U+0153, U+0153, U+0154->U+0155, U+0155, U+0156->U+0157, U+0157, U+0158->U+0159, U+0159, U+015A->U+015B, U+015B, U+015C->U+015D, U+015D, U+015E->U+015F, U+015F, U+0160->U+0161, U+0161, U+0162->U+0163, U+0163, U+0164->U+0165, U+0165, U+0166->U+0167, U+0167, U+0168->U+0169, U+0169, U+016A->U+016B, U+016B, U+016C->U+016D, U+016D, U+016E->U+016F, U+016F, U+0170->U+0171, U+0171, U+0172->U+0173, U+0173, U+0174->U+0175, U+0175, U+0176->U+0177, U+0177, U+0178->U+00FF, U+00FF, U+0179->U+017A, U+017A, U+017B->U+017C, U+017C, U+017D->U+017E, U+017E, U+0410..U+042F->U+0430..U+044F, U+0430..U+044F, U+4E00..U+9FFF 
        min_prefix_len = 0 
        min_infix_len = 0 
    }
    index index_phpbb_[your_id]_delta : index_phpbb_[your_id]_main 
    {
        path = /home/sphinx/index_phpbb_[your_id]_delta 
        source = source_phpbb_[your_id]_delta 
    }
    indexer 
    {
        mem_limit = 1024M 
    }
    searchd 
    {
        compat_sphinxql_magics = 0 
        listen = 127.0.0.1:9312 
        log = /home/sphinx/log/searchd.log 
        query_log = /home/sphinx/log/sphinx-query.log 
        read_timeout = 5 
        max_children = 30 
        pid_file = /home/sphinx/searchd.pid 
        max_matches = 20000 
        binlog_path = /home/sphinx/ 
    }

We specify the server, the username and the password to connect to MySQL, then we copy the whole, which will be used in a few minutes.

Still in the ACP, let's go to `Maintenance` > `Search index` in order to delete the now useless `MySQL Fulltext`, and to create a `Sphinx Fulltext` index. Once done, phpBB displays the number of indexed messages.

## Server-side configuration

Back on our server, we will edit the `sphinx.conf` to paste the previously saved configuration file:

    sudo nano /etc/sphinxsearch/sphinx.conf

Some modifications are however to be made because the configuration file proposed in phpBB 3.2 is a bit old and will cause an error like:

    Restarting sphinxsearch: Sphinx 2.2.9-id64-release (rel22-r5006)
    Copyright (c) 2001-2015, Andrew Aksyonoff
    Copyright (c) 2008-2015, Sphinx Technologies Inc (http://sphinxsearch.com)

    using config file '/etc/sphinxsearch/sphinx.conf'...
    WARNING: key 'sql_query_info' was permanently removed from Sphinx configuration. Refer to documentation for details.
    WARNING: key 'charset_type' was permanently removed from Sphinx configuration. Refer to documentation for details.
    WARNING: key 'max_matches' was permanently removed from Sphinx configuration. Refer to documentation for details.
    ERROR: unknown key name 'compat_sphinxql_magics' in /etc/sphinxsearch/sphinx.conf line 90 col 24.
    FATAL: failed to parse config file '/etc/sphinxsearch/sphinx.conf'

To do this, delete the following lines:

    sql_query_info = SELECT * FROM phpbb_posts WHERE post_id = $id 
    charset_type = utf-8 
    compat_sphinxql_magics = 0 
    max_matches = 20000 

To start *Sphinx* at startup, edit the following file:

    sudo nano /etc/default/sphinxsearch 

... to change the line:

    START=yes

In order to index the data a first time, we use the command:

    sudo indexer --config /etc/sphinxsearch/sphinx.conf index_phpbb_[your_id]_main

Then, to automate the indexing of the site (let's say every minute), we create a `cron` task with `sudo crontab -e` to add:

    indexer --rotate --config /etc/sphinxsearch/sphinx.conf index_phpbb_[your_id]_delta >> /home/sphinx/log/indexer.log 2>&1 &


