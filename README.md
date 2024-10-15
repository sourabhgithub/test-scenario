@Configuration
@EnableBatchProcessing
public class ZellMcJobConfig {

    @Autowired
    private JobBuilderFactory jobBuilderFactory;

    @Autowired
    private StepBuilderFactory stepBuilderFactory;

    @Autowired
    private DataSource dataSource;

    @Autowired
    private EntityManagerFactory entityManagerFactory;

    @Bean
    public Job cashExceptionsProcessJob(JobExecutionListener jobListener, 
                                        Step loadRefDataStep, 
                                        Step zelleMcReconExceptionsStep, 
                                        Step createZelleMcReconExceptionsStep) {
        return jobBuilderFactory.get("cashExceptionsProcessJob")
                .incrementer(new RunIdIncrementer())
                .listener(jobListener)
                .start(loadRefDataStep)
                .next(zelleMcReconExceptionsStep)
                .next(createZelleMcReconExceptionsStep)
                .build();
    }

    @Bean
    public Step loadRefDataStep(PromotionListener promotionListener) {
        return stepBuilderFactory.get("loadRefData")
                .tasklet(loadRefDataTasklet())
                .listener(promotionListener)
                .build();
    }

    @Bean
    public Step zelleMcReconExceptionsStep(PromotionListener promotionListener) {
        return stepBuilderFactory.get("zelleMcReconExceptionsStep")
                .tasklet(zelleMcReconExceptionsTasklet())
                .listener(promotionListener)
                .build();
    }

    @Bean
    public Step createZelleMcReconExceptionsStep(PromotionListener promotionListener) {
        return stepBuilderFactory.get("createZelleMcReconExceptionsStep")
                .tasklet(createZelleMcReconExceptionsTasklet())
                .listener(promotionListener)
                .build();
    }

    @Bean
    public Tasklet loadRefDataTasklet() {
        LoadReferenceDataTasklet tasklet = new LoadReferenceDataTasklet();
        tasklet.setDataSource(dataSource);
        tasklet.setEntityManagerFactory(entityManagerFactory);
        tasklet.setAcctPlanRefSql("batch.service.acct_plan.ref.query");
        tasklet.setHolidayRefSql("batch.service.holiday.ref.query");
        tasklet.setStateInfoRefSql("batch.service.state_info.ref.query");
        tasklet.setTradeTxnRefSql("batch.service.tran_code.ref.query");
        return tasklet;
    }

    @Bean
    public Tasklet zelleMcReconExceptionsTasklet() {
        ProcessReconExceptionsTasklet tasklet = new ProcessReconExceptionsTasklet();
        tasklet.setDataSource(dataSource);
        tasklet.setMissingInCashSql("zelle.mc.recon.missing.in.cash.query");
        tasklet.setMissingInReconTblSql("zelle.mc.recon.missing.in.mc.query");
        tasklet.setAcctPlanRefSql("batch.service.acct_plan.ref.query");
        return tasklet;
    }

    @Bean
    public Tasklet createZelleMcReconExceptionsTasklet() {
        CreateReconExceptionsTasklet tasklet = new CreateReconExceptionsTasklet();
        tasklet.setDataSource(dataSource);
        tasklet.setCashTranExceptInsertSql("visa.mc.cash.tran.except.insert");
        tasklet.setCashTranStatExceptInsertSql("cash.tran.stat.except.insert");
        tasklet.setAcctTranExceptInsertSql("acct.tran.except.insert");
        tasklet.setCashTranStat2InsertSql("cash.tran.stat2.insert");
        tasklet.setCashTranStat2RevInsertSql("cash.tran.stat2.rev.insert");
        tasklet.setAcctEntImpacsExceptInsertSql("acct.ent.impacs.except.insert");
        tasklet.setWipToWipAcctEntInsertSql("wipToWipAcctEntInsertSql");
        return tasklet;
    }

    @Bean
    public PromotionListener promotionListener() {
        PromotionListener promotionListener = new PromotionListener();
        promotionListener.setKeys(Arrays.asList("domainDataKey", "cashPayLoadData", "exceptTranList"));
        return promotionListener;
    }

    @Bean
    public JobExecutionListener jobListener() {
        return new CashExceptionsBatchJobListener();
    }

    @Bean
    public KieBaseManager kieBaseManager() {
        KieBaseManager manager = new KieBaseManager();
        manager.init(); // Initialize manager if needed
        return manager;
    }
}
