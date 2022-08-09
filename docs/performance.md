# [Optimizing your TM1 and Planning Analytics server for Performance](https://code.cubewise.com/blog/optimizing-your-tm1-and-planning-analytics-server-for-performance)

The combination of in-memory calculation, amazing design and years of optimization have resulted in IBM TM1 and Planning Analytics being renowned as one of the fastest real-time analytical engines on the market. The default settings you get "*out of the box*" is all that is required to have a fast TM1 model. This article focuses on taking it a step further using the many parameters (over 100) which allow you to tune your system to get maximum performance of your TM1/Planning Analytics server. 

## MTQ

Multi-threaded querying allows you to enable multiple cores to conduct queries. This feature will provide you with significant performance improvements, especially for large queries with lot of consolidations. An optimal number of cores (sweat spot) needs to be established to achieve the maximum performance. Ensure you conduct various tests to find the “sweet spot” and maximize the performance. Be cautious to not exceed your licensing arrangements. Basically, ensure you have enough PVU licenses:

- [Everything you need to know about MTQ](https://code.cubewise.com/blog/mastering-mtq-with-tm1-and-planning-analytics)

## MTFeeders

[MTFeeders](https://www.ibm.com/support/knowledgecenter/en/SSD29G_2.0.0/com.ibm.swg.ba.cognos.tm1_inst.2.0.0.doc/c_mtfeeders.html) is a new parameter from Planning Analytics (TM1 server v11). By turning on this new parameter in tm1s.cfg, MTQ will be then triggered when recalculating feeders:

- CubeProcessFeeders() is triggered from a TM1 process.

- A feeder statement is updated in the rules.

- Construction of feeders at startup.

MTFeeders will provide you significant improvement but you need to be aware that it does not support conditional feeders. If you are using conditional feeders where the condition clause contains a fed value, you have to turn it off.

To turn on MTFeeders during server start-up you will need to add MTFeeders.AtStartup=T.

## ParallelInteraction (TM1 only)

This feature is turned on but default in Planning Analytics (TM1 11+), you need to set it to true only if you are still using TM1 10.2.

Parallel interaction allows for greater concurrency of read and write operations on the same cube objects. It can be crucial for optimizing lengthy data load processes. Instead of loading all data sequentially, you could load all months at the same time which is called **Parallel loading**. Parallel loading will allow you to segment your data and subsequently leverage multiple cores to simultaneously load the data into cubes:

- [4 ways to execute processes in parallel](https://code.cubewise.com/blog/4-ways-to-speed-up-your-processes-in-ibm-tm1-and-planning-analytics)

## MaximumCubeLoadThreads

This parameter impacts only the start-up time of your PA/TM1 instance. It specifies whether the cube and feeder calculation phases of the server loading are multi-threaded, so multiple processes can be used in parallel. You will need to specify the number of cores that you would like to dedicate to cube loading and feeder processing.

Particularly useful if you have many large cubes and there is an imperative to improve performance upon server start-up. It is recommended that you specify the **maximum amount of cores – 1**.

Similar as for MTQ, to find the optimal number of cores which will provide optimal performance you will need to test multiple scenarios.

However if you are using Planning Analytics, you should use the new parameter **MTCubeLoad** instead. More information on this in this article:

- [Adjusting server configuration for Planning Analytics](http://cubewise.com/blog/adjusting-server-configuration-planning-analytics/)

## PersistentFeeders

Persistent Feeders allows you to improve the loading of cubes with feeders, which will also improve the server start-up time. When you active persistent feeders, it will create a .feeders file for each cube that has rules. Upon server startup the tm1 server will reference the .feeders files and will re-load the feeders for cubes.

It is best practice to activate persistent feeders if you have large cubes which have an extensive number of fed cells.

In many cases start-up time can be significantly reduced, examples of a 80-90% reduction are common.

Things to look out for

- Feeders are saved to the .feeders file. Therefore, even if you remove a particular feeder from the rule file it will remain in the .feeders file. You will need to delete the .feeders file and allow TM1 to re-generate the file.

- Although this is a greater feature, judgment is required on when to use it. For instance, if your cubes are small and don’t have many rules/feeders it may be more beneficial to leave this off.

## Other parameters which will improve user experience

- [AllRuleCalcStargateOptimization](https://www.ibm.com/support/knowledgecenter/SSD29G_2.0.0/com.ibm.swg.ba.cognos.tm1_inst.2.0.0.doc/c_allrulecalcstargateopt.html) can improve performance in calculating views that contain only rule-calculated consolidations.

- [UseStargateForRules](https://www.ibm.com/support/knowledgecenter/en/SSD29G_2.0.0/com.ibm.swg.ba.cognos.tm1_inst.2.0.0.doc/c_usestargateforrules_1.html) By default when retrieving a calculated cell, the value will be retrieved from a stargate view stored in memory, in some unique instances using a stargate view can be slower than requesting the value from the server, so you can turn off Stargate views for rules by using UseStargateForRules=F.

- [ViewConsolidationOptimization](https://www.ibm.com/support/knowledgecenter/en/SSD29G_2.0.0/com.ibm.swg.ba.cognos.tm1_inst.2.0.0.doc/c_viewconsolidationoptimization_1.html) enables or disables view consolidation optimization. It increases the performance but increases the amount of memory required for a given view.

- [CalculationThresholdForStorage](https://www.ibm.com/support/knowledgecenter/SSD29G_2.0.0/com.ibm.swg.ba.cognos.tm1_inst.2.0.0.doc/c_calculation_threshold_for_storage_1.html): The number of cells needed as minimum before stargate view **creation** is triggered. Set to a low number to maximize caching but increase memory.

- [MaximumViewSize](https://www.ibm.com/support/knowledgecenter/SSD29G_2.0.0/com.ibm.swg.ba.cognos.tm1_inst.2.0.0.doc/c_maximumviewsize_1.html): if the view memory when accessing this view reaches the threshold it will abort view construction rather than client waiting forever the view.

- [CheckFeedersMaximumCells](https://www.ibm.com/support/knowledgecenter/SSD29G_2.0.0/com.ibm.swg.ba.cognos.tm1_inst.2.0.0.doc/c_checkfeedersmaximumcells_1.html): if a user tries to check feeders in the cube viewer from cell that is too many cells in the consolidation it will refuse, rather than a very long client hang or eventual crash.

- [MaximumUserSandboxSize](https://www.ibm.com/support/knowledgecenter/SSD29G_2.0.0/com.ibm.swg.ba.cognos.tm1_inst.2.0.0.doc/c_tm1_maxusersandboxsize_parameter.html): Stop server using excessive memory in case users try to do very large sandbox changes

- [LogReleaseLineCount](https://www.ibm.com/support/knowledgecenter/SSD29G_2.0.0/com.ibm.swg.ba.cognos.tm1_inst.2.0.0.doc/c_logreleaselinecount_tm1.html): If admins are doing transaction log queries this stops users getting locked for a long time.

- [StartupChores](https://www.ibm.com/support/knowledgecenter/SSD29G_2.0.0/com.ibm.swg.ba.cognos.tm1_inst.2.0.0.doc/c_tm1_op_startupchores.html): A stargate view is created the first time a user opens the view. If a second a user open the same view, it will be faster because the view would have already been cached. To avoid the first user to wait a bit more, you could set up a chore which ran when the server starts to cache views then this could give users better performance.

- [SubsetElementBreatherCount](https://www.ibm.com/support/knowledgecenter/SSD29G_2.0.0/com.ibm.swg.ba.cognos.tm1_inst.2.0.0.doc/c_subsetelement_breathercount_1.html): Allows lock on subset to be released when there are other requests pending

- [UseLocalCopiesForPublicDynamicSubsets](https://www.ibm.com/support/knowledgecenter/SSD29G_2.0.0/com.ibm.swg.ba.cognos.tm1_inst.2.0.0.doc/c_uselocalcopiesforpublicsubsets_tm1_op_1.html): Improve performance by not invalidating and causing write lock of the public dynamic subset just the user’s local copy.

- [JobQueuing](https://www.ibm.com/support/knowledgecenter/SSD29G_2.0.0/com.ibm.swg.ba.cognos.tm1_inst.2.0.0.doc/c_jobqueuing_cf84006.html): Turns on queuing for Personal Workspace or Sandbox submissions.

- [JobQueueThreadPoolSize](https://www.ibm.com/support/knowledgecenter/SSD29G_2.0.0/com.ibm.swg.ba.cognos.tm1_inst.2.0.0.doc/c_tm1_admn_cfg_jobqueuethreadpoolsize_param.html): Job queue is specific for using contributor/application web which uses sandbox by default. It manages all user sandbox commits into a job queue so users don’t wait.

It is important to be aware that all the parameters in the tm1s.cfg file are now dynamic in IBM Planning Analytics, meaning they can be changed with immediate effect.

## READ MORE:

- [10 tips to improve your TM1 application](https://code.cubewise.com/blog/10-tips-to-improve-your-tm1-application)

- [How to monitor the health of your TM1 server](https://code.cubewise.com/blog/why-its-vital-to-monitor-the-health-of-your-tm1-system)

- [Mastering TM1 with online resources](https://code.cubewise.com/blog/mastering-tm1-with-free-online-resources)
