配置spring batch
依赖

compile("org.springframework.boot:spring-boot-starter-batch")
配置 application.yml

# SPRING BATCH (BatchDatabaseInitializer)
spring:
    batch:
        job:
            enabled: true
        initializer:
            enabled: true
目录大致结构

|-batch
    |- entity
        BaseOrderDo.java
        BusinessDataDo.java
    |- job
        BusinessDataDoJobConf.java
    |- scheduled
        BusinessDataDoScheduledTask.java
    |- step
        |- processor
            BusinessDataDoProcessor.java
        |- step
            BusinessDataDoStepConf.java
    |- utils
        SpringContextUtil.java
下面开始写每一个文件类

BaseOrderDo.java

package com.xunao.rubber.job.batch.entity;

import com.xunao.rubber.domain.enumeration.OrderState;

import javax.persistence.Column;
import java.sql.Timestamp;
import java.time.ZonedDateTime;

/**
 * BaseOrderDo class
 *
 * @author J.Wong
 * @date 2017/11/17
 */
public class BaseOrderDo {

    private Long id;

    @Column(name = "order_state")
    private OrderState orderState;

    private Double price;

    private Double freight;

    @Column(name = "created_date")
    private Timestamp createdDate;

    /******************* getter & setter *******************/

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public OrderState getOrderState() {
        return orderState;
    }

    public void setOrderState(OrderState orderState) {
        this.orderState = orderState;
    }

    public Double getPrice() {
        return price;
    }

    public void setPrice(Double price) {
        this.price = price;
    }

    public Double getFreight() {
        return freight;
    }

    public void setFreight(Double freight) {
        this.freight = freight;
    }

    public Timestamp getCreatedDate() {
        return createdDate;
    }

    public void setCreatedDate(Timestamp createdDate) {
        this.createdDate = createdDate;
    }
}
BusinessDataDo.java

package com.xunao.rubber.job.batch.entity;

import com.xunao.rubber.domain.enumeration.BusinessType;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedBy;
import org.springframework.data.annotation.LastModifiedDate;

import javax.persistence.Column;
import java.time.ZonedDateTime;

/**
 * BusinessDataJob class
 *
 * @author J.Wong
 * @date 2017/11/16
 */
public class BusinessDataDo {

    private Long id;

    @Column(name = "start_time")
    private ZonedDateTime startTime;

    private Integer quantity;

    @Column(name = "total_money")
    private Double totalMoney;

    @Column(name = "business_type")
    private BusinessType businessType;

    @Column(name = "created_by")
    private String createdBy;

    @Column(name = "created_date")
    private ZonedDateTime createdDate = ZonedDateTime.now();

    @Column(name = "last_modified_by")
    private String lastModifiedBy;

    @Column(name = "last_modified_date")
    private ZonedDateTime lastModifiedDate = ZonedDateTime.now();

    /******************* getter & setter *******************/

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public ZonedDateTime getStartTime() {
        return startTime;
    }

    public void setStartTime(ZonedDateTime startTime) {
        this.startTime = startTime;
    }

    public Integer getQuantity() {
        return quantity;
    }

    public void setQuantity(Integer quantity) {
        this.quantity = quantity;
    }

    public Double getTotalMoney() {
        return totalMoney;
    }

    public void setTotalMoney(Double totalMoney) {
        this.totalMoney = totalMoney;
    }

    public BusinessType getBusinessType() {
        return businessType;
    }

    public void setBusinessType(BusinessType businessType) {
        this.businessType = businessType;
    }

    public String getCreatedBy() {
        return createdBy;
    }

    public void setCreatedBy(String createdBy) {
        this.createdBy = createdBy;
    }

    public ZonedDateTime getCreatedDate() {
        return createdDate;
    }

    public void setCreatedDate(ZonedDateTime createdDate) {
        this.createdDate = createdDate;
    }

    public String getLastModifiedBy() {
        return lastModifiedBy;
    }

    public void setLastModifiedBy(String lastModifiedBy) {
        this.lastModifiedBy = lastModifiedBy;
    }

    public ZonedDateTime getLastModifiedDate() {
        return lastModifiedDate;
    }

    public void setLastModifiedDate(ZonedDateTime lastModifiedDate) {
        this.lastModifiedDate = lastModifiedDate;
    }

}
BusinessDataDoJobConf.java

package com.xunao.rubber.job.batch.job;

import org.springframework.batch.core.Job;
import org.springframework.batch.core.Step;
import org.springframework.batch.core.configuration.annotation.EnableBatchProcessing;
import org.springframework.batch.core.configuration.annotation.JobBuilderFactory;
import org.springframework.batch.core.launch.support.RunIdIncrementer;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * BusinessDataDoJobConf class
 *
 * @author J.Wong
 * @date 2017/11/16
 */
// @Configuration
// @EnableBatchProcessing
public class BusinessDataDoJobConf {

    @Autowired
    private JobBuilderFactory jobBuilderFactory;

    @Autowired
    private Step businessDataDoStep;

    @Bean
    public Job businessDataDoJob() {

        return jobBuilderFactory.get("businessDataDoJob")
            .incrementer(new RunIdIncrementer())
            // .listener(listener)
            .flow(businessDataDoStep)
            .end()
            .build();
    }

}
BusinessDataDoScheduledTask.java

package com.xunao.rubber.job.batch.scheduled;

import com.xunao.rubber.job.batch.utils.SpringContextUtil;
import org.springframework.batch.core.Job;
import org.springframework.batch.core.JobParameters;
import org.springframework.batch.core.JobParametersBuilder;
import org.springframework.batch.core.JobParametersInvalidException;
import org.springframework.batch.core.launch.JobLauncher;
import org.springframework.batch.core.repository.JobExecutionAlreadyRunningException;
import org.springframework.batch.core.repository.JobInstanceAlreadyCompleteException;
import org.springframework.batch.core.repository.JobRestartException;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.EnableScheduling;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import java.util.Date;

/**
 * BusinessDataDoScheduledTask class
 *
 * @author J.Wong
 * @date 2017/11/16
 */
// @Component
// @EnableScheduling
public class BusinessDataDoScheduledTask {

    @Autowired
    private JobLauncher jobLauncher;

    @Autowired
    private Job businessDataDoJob;

    // @Scheduled(cron = "0/5 * * * * ?")
    public void reportCurrentTime() {

        System.out.println("jobLauncher: " + jobLauncher);

        JobParameters jobParameters = new JobParametersBuilder()
            .addDate("date", new Date())
            .toJobParameters();

        try {
            jobLauncher.run(businessDataDoJob, jobParameters);
            System.out.println("批处理任务执行完成，date:" + System.currentTimeMillis());
        } catch (JobExecutionAlreadyRunningException e) {
            e.printStackTrace();
        } catch (JobRestartException e) {
            e.printStackTrace();
        } catch (JobInstanceAlreadyCompleteException e) {
            e.printStackTrace();
        } catch (JobParametersInvalidException e) {
            e.printStackTrace();
        }

        System.out.println("jwong-" + Math.random());
    }
}
BusinessDataDoProcessor.java

package com.xunao.rubber.job.batch.step.processor;

import com.xunao.rubber.job.batch.entity.BaseOrderDo;
import com.xunao.rubber.job.batch.entity.BusinessDataDo;
import org.springframework.batch.item.ItemProcessor;

/**
 * BusinessDataDoProcessor class
 *
 * @author J.Wong
 * @date 2017/11/17
 */
public class BusinessDataDoProcessor implements ItemProcessor<BaseOrderDo, BusinessDataDo> {

    @Override
    public BusinessDataDo process(BaseOrderDo item) throws Exception {
        System.out.println(Thread.currentThread().getName()+"process..." + item.toString());
        return null;
    }
}
BusinessDataDoStepConf.java

package com.xunao.rubber.job.batch.step.step;

import com.xunao.rubber.job.batch.entity.BaseOrderDo;
import com.xunao.rubber.job.batch.entity.BusinessDataDo;
import com.xunao.rubber.job.batch.step.processor.BusinessDataDoProcessor;
import org.springframework.batch.core.Step;
import org.springframework.batch.core.configuration.annotation.EnableBatchProcessing;
import org.springframework.batch.core.configuration.annotation.StepBuilderFactory;
import org.springframework.batch.core.configuration.annotation.StepScope;
import org.springframework.batch.item.ItemProcessor;
import org.springframework.batch.item.database.BeanPropertyItemSqlParameterSourceProvider;
import org.springframework.batch.item.database.JdbcBatchItemWriter;
import org.springframework.batch.item.database.JdbcPagingItemReader;
import org.springframework.batch.item.database.Order;
import org.springframework.batch.item.database.support.MySqlPagingQueryProvider;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jdbc.core.BeanPropertyRowMapper;

import javax.sql.DataSource;
import java.time.ZonedDateTime;
import java.time.format.DateTimeFormatter;
import java.util.HashMap;

/**
 * BusinessDataDoStepConf class
 *
 * @author J.Wong
 * @date 2017/11/16
 */
// @Configuration
// @EnableBatchProcessing
public class BusinessDataDoStepConf {

    @Autowired
    private StepBuilderFactory stepBuilderFactory;

    @Autowired
    private DataSource dataSource;

    // @Bean
    public Step businessDataDoStep() {
        return stepBuilderFactory.get("businessDataDoStep")
            .allowStartIfComplete(true)
            // .listener(steplistener)
            .<BaseOrderDo, BusinessDataDo>chunk(10)
            .reader(reader())
            .processor(process())
            .writer(writer())
            .build();
    }

    // @Bean
    // @StepScope
    public JdbcPagingItemReader<BaseOrderDo> reader() {

        // 获取当前时间
        ZonedDateTime nowTemp = ZonedDateTime.now();
        ZonedDateTime yesterdayEnd = ZonedDateTime.of(nowTemp.getYear(), nowTemp.getMonthValue(), nowTemp.getDayOfMonth(), 0, 0, 0, 0, nowTemp.getZone());
        // ZonedDateTime yesterdayStart = yesterdayEnd.minusDays(1);
        ZonedDateTime yesterdayStart = yesterdayEnd.minusDays(1);

        System.out.println("yesterdayStart: " + yesterdayStart);
        System.out.println("yesterdayEnd: " + yesterdayEnd);

        //创建Reader
        JdbcPagingItemReader<BaseOrderDo> reader = new JdbcPagingItemReader<>();
        //查询起始执行行数
        reader.setDataSource(dataSource);
        reader.setFetchSize(0);

        reader.setRowMapper(new BeanPropertyRowMapper<>(BaseOrderDo.class));
        reader.setQueryProvider(new MySqlPagingQueryProvider() {
            {
                setSelectClause("select id, order_state, price, freight, created_date");
                setFromClause("from base_order");
                setWhereClause("created_date >= :createdDate1 and created_date < :createdDate2");
                setSortKeys(new HashMap<String, Order>() {
                    {
                        put("id", Order.ASCENDING);
                    }
                });
            }
        });
        reader.setParameterValues(new HashMap<String, Object>() {
            {
                put("createdDate1", yesterdayStart.format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
                put("createdDate2", yesterdayEnd.format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
            }
        });
        System.out.println("read...");

        return reader;
    }

    // @Bean
    // @StepScope
    public ItemProcessor<BaseOrderDo, BusinessDataDo> process() {
        return new BusinessDataDoProcessor();
    }

    // @Bean
    // @StepScope
    public JdbcBatchItemWriter<BusinessDataDo> writer() {
        JdbcBatchItemWriter<BusinessDataDo> writer = new JdbcBatchItemWriter<>();
        writer.setDataSource(dataSource);
        writer.setItemSqlParameterSourceProvider(new BeanPropertyItemSqlParameterSourceProvider<>());
        writer.setSql("insert into business_data" +
            "(id, start_time, quantity, total_money, business_type, created_by) " +
            "value " +
            "(:id, :startTime, :quantity, :totalMoney, :businessType, :createdBy)");
        System.out.println("write...");
        return writer;
    }

}

SpringContextUtil.java

package com.xunao.rubber.job.batch.utils;

import javax.servlet.ServletContext;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.web.context.ServletContextAware;

/**
 * SpringContextUtil class
 * <p>
 * 从Spring容器中取得对象
 *
 * @author J.Wong
 * @date 2017/11/16
 */
public class SpringContextUtil implements ApplicationContextAware,
    ServletContextAware {

    private static ApplicationContext applicationContext; // Spring上下文对象.静态变量,可在任何代码任何地方任何时候中取出ApplicaitonContext.

    private static ServletContext servletContext;// 注入 系统上下文对象

    Log log = LogFactory.getLog(SpringContextUtil.class);

    /**
     * 实现ApplicationContextAware接口的回调方法，设置上下文环境
     *
     * @param applicationContext
     * @throws BeansException
     */
    public void setApplicationContext(ApplicationContext applicationContext) {
        log.debug(" com.hna.hka.rmc.common.util.SpringContextUtil setApplicationContext " + applicationContext);
        SpringContextUtil.applicationContext = applicationContext;
    }

    /**
     * @return ApplicationContext
     */
    public static ApplicationContext getApplicationContext() {
        return applicationContext;
    }

    /**
     * 获取对象
     *
     * @param name
     * @return Object 一个以所给名字注册的bean的实例
     * @throws BeansException
     */
    public static Object getBean(String name) throws BeansException {
        return applicationContext.getBean(name);
    }

    /**
     * 功能 : 实现 ServletContextAware接口,由Spring自动注入 系统上下文对象
     **/
    public void setServletContext(ServletContext servletContext) {
        SpringContextUtil.servletContext = servletContext;
    }

    /**
     * @return ServletContext
     */
    public static ServletContext getServletContext() {
        return servletContext;
    }
}
