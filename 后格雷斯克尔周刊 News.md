## PostgreSQL Weekly News - May 30, 2021
[https://www.postgresql.org/about/news/postgresql-weekly-news-may-30-2021-2228/](https://www.postgresql.org/about/news/postgresql-weekly-news-may-30-2021-2228/)

PostgreSQL每周新闻-2021年5月30号
pgSCV,一个与Prometheus兼容的监控代理和用于PostgreSQL的度量导出程序，已发布。
Pgpool-II 4.2.3, 4.1.7, 4.0.14, 3.7.19 and 3.6.26,已发布一个用于PostgreSQL的连接池和语句复制系统。
sqlite_fdw 1.2.0 ,已发布。
Crunchy PostgreSQL Operator 4.7.0,已发布一个在Kubernetes上部署和管理开源PostgreSQL集群的系统。
pgAdmin4 5.3,已发布一个用于PostgreSQL的web和本地GUI控制中心。
InfluxDB fdw 1.0.0 发布 https://github.com/pgspider/influxdb_fdw
griddb_fdw 2.0 发布。

PostgreSQL 产品新闻

PostgreSQL新闻

Planet PostgreSQL: https://planet.postgresql.org/
PostgreSQL每周新闻由David Fetter提供给您
请于周日下午3:00 (PST8PDT)前将新闻和公告提交至david@fetter.org。

应用补丁
David Rowley pushed:
.在构建结果缓存路径时添加缺失的NULL检查。在9e215378d中添加的代码，当不是所有连接条件都是唯一连接参数化的一部分时，不能在检查param_info的ppi_clauses之前首先检查内部路径的param_info是否设置了。在这里添加一个NULL值检查，如果param_info为NULL，就不要尝试构建路径。当决定连接是否唯一时，不会考虑Lateral_vars，所以当有Lateral_vars而没有param_info时，我们不会错过进行优化。Reported-by:Coverity, via Tom Lane Discussion:https://postgr.es/m/457998.1621779290@sss.pgh.pa.us https://git.postgresql.org/pg/commitdiff/99c5852e20a0987eca1c38ba0c09329d4076b6a0

.修复结果缓存节点的setrefs.c代码。在9eacee2e6中添加的结果缓存忽略了正确调整setrefs.c中的计划引用。这可能导致以下错误在EXPLAIN: error: cannot decompile join alias var in plan tree修复。Bug: 17030 Reported-by: Hans Buschmann Discussion: https://postgr.es/m/17030-5844aecae42fe223@postgresql.org https://git.postgresql.org/pg/commitdiff/cba5c70b956810c61b3778f7041f92fbb8065acb

Tom Lane pushed:
.DOC:将一些catalogs.sgml条目移到正确的位置,pg_statistic_ext_data.stxdexpr和pg_stats_ext.exprs被列在错误的目录下，另外还有一个pg_statistic_ext_data.stxexprs的伪条目，显然是提交a4d75c86b中的合并失败。Guillaume Lelarge and Tom Lane Discussion:https://postgr.es/m/CAECtzeUHw+w64eUFVeV_2FJviAw6oZ0wNLkmU843ZH4hAQfiWg@mail.gmail.com https://git.postgresql.org/pg/commitdiff/713a431c781fbfe1a22fae4991836077f0f4c513

.修复inline_function()中未初始化变量的使用。Commit e717a9a18引入了一个绕过get_expr_result_type调用的代码路径，这样并不好，因为我们需要将其retupdesc结果传递给check_sql_fn_retval。我们没有立即注意到，因为check_sql_fn_retval使用该参数的代码路径在这种上下文中很难找到。但这并不是不可能的，而且在任何情况下，假设check_sql_fn_retval不需要这个值，inline_function都没有任何意义。要解决这个问题，需要将get_expr_result_type移出if-block，这反过来需要将虚拟FuncExpr的构造移出if-block。 Per report from Ranier Vilela. Discussion: https://postgr.es/m/CAEudQAqBqQpQ3HruWAGU_7WaMJ7tntpk0T8k_dVtNB46DqdBgw@mail.gmail.com https://git.postgresql.org/pg/commitdiff/e30e3fdea873e4e9517c490232ea1d3bcef6c643

.重新考虑pg_attribute.attcompression的定义。将'\0' (InvalidCompressionMethod)重新定义为“如果我们需要压缩，使用default_toast_compression的当前设置”。这使得'\0'成为一个合适的默认选择，而不管数据类型怎么样，极大地简化了初始化tuple descs 等的代码路径。这似乎是一种更方便用户的方法，因为现在默认的压缩选择不会迁移到表定义中，这意味着更改default_toast_compression通常足以改变安装的行为;无需繁琐地发出每列ALTER SET COMPRESSION命令。在此过程中，修复每列压缩特性的一些小bug和文档问题。为SetIndexStorageProperties和GetAttributeCompression采用更健康的api。Bump catversion是因为attcompression的典型内容现在将会不同。我们可以不这样做，但最好确保v14安装都同意这一点。（反正我们已经为 beta 2 强制了 initdb )。Discussion:     https://postgr.es/m/626613.1621787110@sss.pgh.pa.us https://git.postgresql.org/pg/commitdiff/e6241d8e030fbd2746b3ea3f44e728224298f35b

.减少为genbki.pl保留的OID 范围。提交ab596105b将FirstBootstrapObjectId从12000增加到13000，但我们对此有一些调整。令人担忧的是减少there和FirstNormalObjectId之间的时间，因为在initdb期间为排序对象消耗的OID 数量是很难预测的。我们可以通过放弃这些OID 必须是全局唯一的假设来改善这种情况。对于每个目录都是唯一的就足够了。(任何对此不满意的代码都会被破坏，因为一旦OID计数器结束，就不能保证超过每个目录的唯一性。)通过这个更改，在genbki.pl期间分配的最大OID(从10000开始)略小于。

.这使得FirstBootstrapObjectId恢复到12000，并有足够的信心在未来的许多年内保持足够。目前，我们并没有放弃手动分配的OID（低于10000个）是全球唯一的这一期望。总有一天，这可能是必要的，但需要似乎还需要几年。对于v14来说，这已经晚了，但是现在这样做似乎是值得的，这样下游软件就不必处理FirstBootstrapObjectId更改的后果。在任何情况下，我们已经同意为beta2强制使用initdb，所以在增加一个catversion bump不会有什么坏处。Discussion: https://postgr.es/m/1665197.1622065382@sss.pgh.pa.us https://git.postgresql.org/pg/commitdiff/a4390abecf0f5152cff864e82b67e5f6c8489698

.Doc：改进libpq服务文件docs，避免过度指定路径名。阐明libpq.sgml对服务文件位置和语义的描述。避免使用反勾的pg_config调用来描述路径;这在Windows上是行不通的，即使在Unix上，也不是所有读者都能立即熟悉的。不要过度指定include文件的位置，而只编写在include指令中使用的文件。根据“postgresql”在安装路径中的位置，这些地方的前一段文字对于某些安装是不正确的。我们引用用户主目录的约定似乎是“~”，因此更改拼写为“$home”的位置。install-windows.sgml遵循平台惯例，将文件路径拼写为“\”，因此请更改使用“/”的位置。Haiying Tang and Tom Lane Discussion: https://postgr.es/m/162149020918.26174.7150424047314144297@wrigleys.postgresql.org https://git.postgresql.org/pg/commitdiff/ba356a397de565c014384aa01a945aab7d50928c

Peter Geoghegan pushed:
.考虑在扫描期间触发VACUUM故障保护。提交1e55e7d1添加的环绕式故障安全机制通过添加专用的故障安全检查来处理一次性策略情况(即“表没有索引”情况)。这弥补了lazy_vacuum_all_indexes()中通常的一次遍历检查在一次遍历策略VACUUM中永远无法达到的事实。这种方法没有考虑到选择预先不进行索引清除的两次真空。INDEX_CLEANUP off是唯一能这样工作的情况。通过在第一次扫描堆期间每4GB执行一次故障保险检查来修复这个问题，而不考虑VACUUM的细节。这消除了特殊情况，并将使故障安全触发更可靠。作者：Peter Geoghegan pg@bowt.ie 报告人：Andres Freund andres@anarazel.de 审核人：Masahiko Sawada Sawada.mshk@gmail.com Discussion:https://postgr.es/m/20210424002921.pb3t7h6frupdqnkp@alap3.anarazel.de https://git.postgresql.org/pg/commitdiff/c242baa4a831ac2e7dcaec85feb410aefa3a996e

.修复VACUUM VERBOSE的LP_DEAD项页输出。在提交5100010e中的疏忽。https://git.postgresql.org/pg/commitdiff/9afdea982420f9672b88e5c17d1ee8eec64105fc

Michaël Paquier pushed:
.不允许SSL重新谈判。SSL重新协商在48d23c72时已经被禁用，但是这并不阻止服务器遵从客户端愿意使用重新协商。在过去的几年里，重新协商存在一系列安全问题和缺陷(比如最近的cve - 21-3449)，如果客户机试图重新协商，可能会导致后端崩溃。此提交需要采取一个额外的步骤，即以与SSL压缩(f9264d15)或票证(97d3a0b0)相同的方式禁用后端重新协商。OpenSSL 1.1.0h增加了一个名为SSL_OP_NO_RENEGOTIATION的选项来实现这一点。在旧版本有一个选项称为SSL3_FLAGS_NO_RENEGOTIATE_CIPHERS的选项,，它没有文档记录，可以在TLS连接打开时创建的SSL对象中设置，但我决定不使用它，因为依赖它感觉更困难，而且它不是官方的。注意，这个选项在OpenSSL < 1.1.0h中不可用，因为*SSL对象的内部内容对应用程序是隐藏的。SSL重新协商涉及到TLSv1.2之前的协议。根据Robert Haas的原始报告，根据Andres Freund的建议做了一个补丁。Author: Michael Paquier Reviewed-by: Daniel Gustafsson Discussion: https://postgr.es/m/YKZBXx7RhU74FlTE@paquier.xyz Backpatch-through: 9.6 https://git.postgresql.org/pg/commitdiff/01e6f1a842f406170e5f717305e4a6cf0e84b3ee

.修复在VACUUM FULL/CLUSTER中去解除压缩值拷贝时的内存泄漏。如果当前存储的值是用与列定义的方法不匹配的方法压缩的，那么可以使用VACUUM FULL和CLUSTER强制使用toastable列的现有压缩方法。在重写时，负责解压缩和重新压缩toast值的代码会留在重新压缩的值周围，导致在TopTransactionContext中分配的内存积累。当处理大型关系时，这可能会导致系统耗尽内存。一旦重写了它们的元组，就不需要重新生成的值，并且这种提交确保了必要的清理工作的发生。bbe0a81d引入的问题。该区域的评论是重新排序了一点，而它上。报告作者:Andres Freund分析作者:Michael Paquier评论作者:Dilip Kumar Discussion:https://postgr.es/m/20210521211929.pcehg6f23icwstdb@alap3.anarazel.de https://git.postgresql.org/pg/commitdiff/fb0f5f0172edf9f63c8f70ea9c1ec043b61c770e

.修正了heap .c中的输入错误。Author: Hou Zhijie Discussion:https://postgr.es/m/OS0PR01MB571612191738540B27A8DE5894249@OS0PR01MB5716.jpnprd01.prod.outlook.com https://git.postgresql.org/pg/commitdiff/190fa5a00a8f9ecee8eef2c8e26136b772b94e19

.doc:修复了docs和postgresql.conf.sample中一些GUCs的描述。以下参数的描述不精确或不正确(PGC_POSTMASTER或PGC_SIGHUP):——autovacuum_work_mem(文档,如9.6 ~)——huge_page_size(文档,14 ~)——max_logical_replication_workers(文档,截至10 ~)——max_sync_workers_per_subscription(文档,截至10 ~)——min_dynamic_shared_memory(文档,14 ~)——recovery_init_sync_method (postgresql.conf。样本,14 ~)——remove_temp_files_after_crash(文档,14 ~)——restart_after_crash(文档,在9.6 ~)——ssl_min_protocol_version(文档,12 ~)——ssl_max_protocol_version(文档,12 ~)这个提交调整这些参数的描述更符合实践用于别人。 Revewed-by: Justin Pryzby Discussion:https://postgr.es/m/YK2ltuLpe+FbRXzA@paquier.xyz Backpatch-through: 9.6 https://git.postgresql.org/pg/commitdiff/2941138e60fc711bd221b3264807f36cc079dfbb

.修复使用GSSAPI/Kerberos构建时的MSVC脚本。Windows上的上游Kerberos可交付版本安装的路径与我们的MSVC脚本不匹配。首先，include文件夹在我们的脚本中被命名为“inc/”，但是上游msi使用“include/”。其次，在64位环境中构建会失败，因为库的命名不同。这个提交调整MSVC脚本，以兼容最新安装的上游，我已经检查编译能够与32位和64位安装工作。特别感谢Kondo Yuta帮助调查hamerkop的情况，它有一个错误的GSS编译配置。你们报道:Brian Ye Discussion: https://postgr.es/m/162128202219.27274.12616756784952017465@wrigleys.postgresql.org Backpatch-through: 9.6 https://git.postgresql.org/pg/commitdiff/025110663448a8c877f4b591495f2e5d187d8936

Amit Kapila pushed:
.改进并行vacuum文档和错误消息。错误消息、文档和其中一个选项是使用“并行度”来表示vacuum命令使用的并行度。我们通常在其他地方使用“平行工人”，所以相应地把它换成平行真空。作者:Bharath Rupireddy评论:Dilip Kumar, Amit Kapila Discussion: https://postgr.es/m/CALj2ACWz=PYrrFXVsEKb9J1aiX4raA+UBe02hdRp_zqDkrWUiw@mail.gmail.com https://git.postgresql.org/pg/commitdiff/0734b0e983443882ec509ab4501c30ba9b706f5f

.Doc:更新逻辑解码统计信息。添加pg_stat_replication_slots视图的信息以及其他与逻辑解码相关的系统目录。作者:Vignesh C评论:Amit Kapila Discussion: https://postgr.es/m/20210319185247.ldebgpdaxsowiflw@alap3.anarazel.de https://git.postgresql.org/pg/commitdiff/0c6b92d9c6fb74255467573fde54f65139b26603

.修复多插入toast更改流期间的断言。在解码多重插入的WAL时，在得到WAL记录的最后一个插入之前，我们不能清除toast。现在，如果我们在获得最后一个更改之前对更改进行流处理，那么toast块的内存将不会被释放，我们希望txn在流式处理之后对所有更改进行流式处理。这种限制主要是为了确保流式事务的正确性，仅仅为了允许这种情况而提升这种限制似乎是不值得的，因为无论如何，一旦这样的插入完成，我们将流式处理事务。以前我们使用两个不同的标志（一个用于toast元组，另一个用于推测性插入）来表示部分更改。现在，我们用一个标志替换了这两个标志，以指示部分更改。报告人：Pavan Deolasee作者：Dilip Kumar审查人：Pavan Deolasee，Amit Kapila Discussion: https://postgr.es/m/CABOikdN-_858zojYN-2tNcHiVTw-nhxPwoQS4quExeweQfG1Ug@mail.gmail.com https://git.postgresql.org/pg/commitdiff/6f4bdf81529fdaf6744875b0be99ecb9bfb3b7e0

Peter Eisentraut pushed:
 postgresql.conf.sample:使垂直间距一致。  https://git.postgresql.org/pg/commitdiff/8673a37c85fef00dd5b9c04197538142bec10542
 
 用assertion替换运行时错误检查。错误消息检查从解析器返回的结构是否符合预期。这是我们通常使用认定的东西，而不是面向用户的完整错误消息。所以用assertion替换它(隐藏在lfirst_node()中)。审核:Tom Lane tgl@sss.pgh.pa.us  Discussion: https://www.postgresql.org/message-id/flat/452e9df8-ec89-e01b-b64a-8cc6ce830458%40enterprisedb.com
 
 添加NO_INSTALL选项到pgxs。在libpq_pipeline中应用test makefile，这样测试文件就不会安装到tmp_install中。审核人:Alvaro Herrera alvherre@alvh.no-ip.orgTom Lane tgl@sss.pgh.pa.us Discussion: https://www.postgresql.org/message-id/flat/cb9d16a6-760f-cd44-28d6-b091d5fb6ca7%40enterprisedb.com https://git.postgresql.org/pg/commitdiff/6abc8c2596dbfcb24f9b4d954a1465b8015118c3
 
 修复了libpq_pipeline测试中的vpath构建。需要将路径设置为指向构建目录，而不是当前目录，因为当前目录实际上是此时的源目录。解决6 abc8c2596dbfcb24f9b4d954a1465b8015118c3 https://git.postgresql.org/pg/commitdiff/a717e5c771610cf8607f2423ab3ab6b5d30f44ea
 
Álvaro Herrera pushed:
 降低detach-partition-concurrently-3的时间敏感性。当向等待锁的会话发送一个取消时，这个最近添加的测试对时间太敏感了。我们通过在cancel后立即在阻塞会话中运行一个no-op查询来解决这个问题;这避免了发送cancel的会话在报告cancel之前立即发送另一个查询。Noah Misch的主意。通过这个修复，我们有时会看到cancel错误报告仅相对于被取消的步骤显示，而不是与发送cancel的步骤一起显示。为了增加两个步骤同时显示的可能性，在cancel中添加0.1s sleep。在正常情况下，这似乎足以平息大多数失败，但我们将看到较慢的buildfarm成员对此进行了说明。报告人：Takamichi Osumi osumi.takamichi@fujitsu.com Discussion: https://postgr.es/m/OSBPR01MB4888C4ABA361C7E81094AC66ED269@OSBPR01MB4888.jpnprd01.prod.outlook.com https://git.postgresql.org/pg/commitdiff/5e0b1aeb2dfed4f1eb7ac5154c1573885a70db41
  
  使detach-partition-concurrently-4对时间不那么敏感。与5e0b1aeb2dfe相同，用于附带测试文件。这一次似乎概率较低（一个月内只有两次失败）；如果没有补丁，我几乎无法重现一个故障，所以我也无法用它重现一个故障，这并不说明什么问题。我们必须等待进一步的建设结果，看看我们是否需要进一步的调整。 Discussion: https://postgr.es/m/20210524090712.GA3771394@rfd.leadboat.com https://git.postgresql.org/pg/commitdiff/eb43bdbf5104c183412aac0fccf8e515e60d9212
  
Andrew Dunstan pushed:
  在MSVC 构建 pg_config中报告已配置的端口。这是一个长期存在的遗漏，是在尝试编写依赖于它的代码时发现的。Backpatch到所有活的分支。https://git.postgresql.org/pg/commitdiff/fb424ae85f6b1e32e545f13902d3ba3429be44df
   
  修复语法错误。https://git.postgresql.org/pg/commitdiff/d69fcb9caef1ac1f38241645d4fb9f7e0ce02a70
   
Thomas Munro pushed:
 修复共享元组描述符时的竞争条件。同时为相同的元组描述符调用BlessTupleDesc()的并行查询进程可能会崩溃。有代码可以处理这种罕见的情况，但它解除了对伪DSA指针的引用。修复。回到11，其中提交cc5f8136增加了对并行查询中共享元组描述符的支持。报道: Eric Thinnes e.thinnes@gmx.de Discussion: https://postgr.es/m/99aaa2eb-e194-bf07-c29a-1a76b4f2bcf9%40gmx.de https://git.postgresql.org/pg/commitdiff/b1d6538903504904db44679355ac109aeda01737
 
待定补丁
 Fabien COELHO和Aleksander Alekseev交换了补丁，用更适合64位时代的东西取代了rand48伪随机数生成器。
 Greg Nancarrow和Pavel Borisov交换补丁来修复一个并行工作程序失败的断言和coredump。
 Vigneshwaran C发送了另一个补丁版本，为发布添加了模式级支持。
 Hou Zhijie和Amit Langote交换补丁，在分区键为常数的情况下跳过分区元组路由。
 Dilip Kumar, Tsutomu Yamada和Kyotaro HORIGUCHI在恢复中交换补丁来修复比赛状态。
 Hou Zhijie又发来了一个补丁的修订版，让INSERT并行化成为可能。选择
 Tom Lane发送了两个修订版的补丁，以修复CALL和只带输出参数的过程之间的不一致性。
 Justin Pryzby发送了另一个修订版的补丁，使WAL压缩方法可插入并默认为lz4。
 当root->simple_rte_array时Andy Fan发送了一个补丁来使用planner_rt_fetch而不是rt_fetch。
 Ajin Cherian又发送了5个修订版的补丁，以跳过空事务进行逻辑复制。
 Mark Dilger 又更新了一个补丁，将超级用户任务委派给新的安全角色。
 Hou Zhijie,Bharath Rupireddy和Tomáš Vondra交易补丁，以确保postgres_fdw批处理不会使用太多参数。
 Bharath Rupireddy发送了一个补丁，提供TDE nonce size作为initdb选项，将TDE nonce字节添加到pd_specia特殊结构中，并调整测试以考虑可配置的TDE nonce size。
 Bharath Rupireddy发送了另一个修订版的补丁来消除使用“non-negative”的错误信息的歧异。
 Antonin Houska发送了一个补丁来缩小并发UPDATE重启heap_lock_tuple()的情况，减少不必要的调用。
 Greg Sabino Mullane发送了一个补丁来加快pg_checksum的速度，在校验和已经设置的情况下，避免在校验和已经设置为预期值时写入相同的内容。
 Michaël Paquier发送了一个补丁，修复在使用2PC时提升热备节点时出现的错误快照。
 Bharath Rupireddy发送了另外两个版本的补丁，检查是否有重复的选项，如果在CREATE COLLATION中发现，就会出错。
 Andrey V. Lepikhov发送了一个补丁，教优化器考虑非分区表与分区表的每个分区的分区连接。
 Andrey V. Lepikhov发送了另一个版本的补丁，通过一个新的等价类删除不必要的自连接。
 Peter Eisentraut发送了一个补丁来修复hba文件解析中的RADIUS错误报告。
 Vigneshwaran C和Bharath Rupireddy交换了补丁来改进发布错误消息。
 Tom Lane发送了另一个补丁版本，用固定范围检查替换pg_depend PIN项。
 Kyotaro HORIGUCHI 提交了两份补丁的修订版，将令人困惑的‘bracket’用法改为更清晰的措辞，并添加了用于追踪(多)范围类型垃圾的测试用例。
 Bharath Rupireddy和 Hou Zhijie 交换了补丁，使得在CREATE TABLE AS中使用并行插入成为可能。
 Dilip Kumar 发了一个补丁来修复toast解码推测性插入时的内存泄漏。
 Peter Geoghegan给Generalize VACUUM的INDEX_CLEANUP选项打了一个补丁，允许用户禁用由commit 5100010e添加的索引真空绕过优化，以及将来可能添加的任何类似的优化。
 Paul Guo在另一个版本的补丁中只对受影响的文件/目录进行fsync，并在pg_rewind中使用copy_file_range()进行文件复制。
 Takamichi Osumi发送了另一个补丁的修订版，以记录长时间运行的查询计划。
 Daniel Gustafsson发送了另一个版本的补丁来支持NSS作为libpq TLS后端。
 Etsuro Fujita发送了一个补丁来修复PostgreSQL FDW中异步附加重新扫描。
 Vigneshwaran C发送了一个补丁来添加别名类型regpublication和regsubscription。
 Fabien COELHO发送了一个补丁来减少psql回显代码的重复。
 Laurenz Albe在另一个版本的补丁中扩展了PostgreSQL扩展编码和后台工作者开发的文档，以便像分配、中断处理、退出回调、事务回调、PG_TRY()/PG_CATCH()、资源所有者、事务和快照状态等关键主题，等等，至少简要提到了一些指针，以了解更多。
 Tom Lane发送了一个补丁，以减少待处理的inval消息的内存消耗。
 Andreas Karlsson 给gisstate发了个补丁。
 Yura Sokolov 发了一个补丁来清除页面中的空白。
 Tomáš Vondra发送了一个补丁来恢复COPY FREEZE的部分改进，即调整heap_multi_insert，并删除39b66a91bd的大部分(除了heap_xlog_multi_insert位)。
 Thomas Munro发送了两个版本的补丁，以支持macOS上的直接I/O。
