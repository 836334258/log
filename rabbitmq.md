# php

```php
<?php


namespace App\Services;


use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Exchange\AMQPExchangeType;
use PhpAmqpLib\Message\AMQPMessage;

class RabbitmqService
{
    protected $connection;

    protected $channel;

    protected $queue = 'hello';

    const HOST = '127.0.0.1';
    const PORT = 5672;
    const USER = 'guest';
    const PASSWORD = 'guest';

    const EXCHANGE1='exchange1';
    const EXCHANGE2='exchange2';
    const EXCHANGE_TOPIC='exchange_topic';


    public function __construct()
    {
        $this->connection = new AMQPStreamConnection(self::HOST, self::PORT, self::USER, self::PASSWORD);
        $this->channel = $this->connection->channel();
        //告诉RabbitMQ一次不要给工人一个以上的消息
//        $this->channel->basic_qos(null,1,null);
    }

    /**
     * 简单发送
     */
    public function basicPublish()
    {
        $this->channel->queue_declare("queue1", false, false, false, false);
        // 设置持久消息
        $msg = new AMQPMessage(json_encode(request()->all(),JSON_UNESCAPED_UNICODE),[
            'delivery_mode'=>AMQPMessage::DELIVERY_MODE_PERSISTENT
        ]);
        $this->channel->basic_publish($msg, '', 'queue1');

    }

    /**
     * 简单接收
     *
     * @throws \ErrorException
     */
    public function basicReceiving()
    {
        $this->channel->queue_declare("queue1", false, false, false, false);
        $this->channel->basic_consume("queue1", '', false, true
            , false, false, function ($msg) {
                var_dump($msg->body);
            });

        while ($this->channel->is_consuming()) {
            $this->channel->wait();
        }
    }

    /**
     * 发布订阅模式的发布
     */
    public function publish()
    {
        $this->channel->queue_declare('queue2',false,false,false);
        $this->channel->exchange_declare(self::EXCHANGE1,'fanout',false,false,false);
        $msg=new AMQPMessage(json_encode_512(request()->all()));

        $this->channel->basic_publish($msg,self::EXCHANGE1);

        return 'ok';
    }

    /**
     * 发布订阅模式的接收
     */
    public function receive()
    {
        $this->channel->queue_declare('queue2',false,false,false);
        $this->channel->exchange_declare(self::EXCHANGE1,AMQPExchangeType::FANOUT,false,false,false);
        $this->channel->queue_bind('queue2',self::EXCHANGE1);

        $this->channel->basic_consume("queue2", '', false, true
            , false, false, function ($msg) {
                var_dump($msg->body);
            });

        while ($this->channel->is_consuming()) {
            $this->channel->wait();
        }
    }

    /**
     * 发布订阅模式的接收
     */
    public function receive2()
    {
        $this->channel->queue_declare('queue3',false,false,false);
        $this->channel->exchange_declare(self::EXCHANGE1,AMQPExchangeType::FANOUT,false,false,false);
        $this->channel->queue_bind('queue3',self::EXCHANGE1);

        $this->channel->basic_consume("queue3", '', false, true
            , false, false, function ($msg) {
                var_dump($msg->body);
            });

        while ($this->channel->is_consuming()) {
            $this->channel->wait();
        }
    }

    /**
     * direct 发布
     *
     * @return string
     */
    public function directPublish()
    {
        $this->channel->queue_declare('queue4',false,false,false);
        $this->channel->exchange_declare(self::EXCHANGE2,AMQPExchangeType::DIRECT,false,false,false);
        $msg=new AMQPMessage(json_encode_512(request()->all()));

        $this->channel->basic_publish($msg,self::EXCHANGE2,'aa');

        return 'ok';
    }

    /**
     * direct 接收
     *
     * @throws \ErrorException
     */
    public function directReceive()
    {
        $this->channel->queue_declare('queue4',false,false,false);
        $this->channel->exchange_declare(self::EXCHANGE2,AMQPExchangeType::DIRECT,false,false,false);
        $this->channel->queue_bind('queue4',self::EXCHANGE2,'aa');

        $this->channel->basic_consume("queue4", '', false, true
            , false, false, function ($msg) {
                var_dump($msg->body);
            });

        while ($this->channel->is_consuming()) {
            $this->channel->wait();
        }
    }

    /**
     *  topic 发布
     */
    public function topicPublish()
    {
        $this->channel->queue_declare("queue5");
        $this->channel->queue_declare("queue6");
        $this->channel->exchange_declare(self::EXCHANGE_TOPIC,AMQPExchangeType::TOPIC,false,false);
        $msg=new AMQPMessage(json_encode_512(request()->all()));

//        $route_key='anonymous.info';
        $route_key='anonymous.a.c';

        $this->channel->basic_publish($msg,self::EXCHANGE_TOPIC,$route_key);
    }

    /**
     * topic 接收
     *
     * @throws \ErrorException
     */
    public function topicReceive()
    {
        $this->channel->exchange_declare(self::EXCHANGE_TOPIC,AMQPExchangeType::TOPIC,false,false);
        $this->channel->basic_consume("queue5", '', false, true
            , false, false, function ($msg) {
                var_dump($msg->body);
            });

        $this->channel->queue_bind("queue5",self::EXCHANGE_TOPIC,'anonymous.a.*');
        while ($this->channel->is_consuming()) {
            $this->channel->wait();
        }
    }

    /**
     * topic 接收2
     *  # 代表多个单词，* 代表一个单词
     *
     * @throws \ErrorException
     */
    public function topicReceive2()
    {
        $this->channel->exchange_declare(self::EXCHANGE_TOPIC,AMQPExchangeType::TOPIC,false,false);
        $this->channel->basic_consume("queue6", '', false, true
            , false, false, function ($msg) {
                var_dump($msg->body);
            });

        $this->channel->queue_bind("queue6",self::EXCHANGE_TOPIC,'anonymous.#');
        while ($this->channel->is_consuming()) {
            $this->channel->wait();
        }
    }



    public function __destruct()
    {
        // TODO: Implement __destruct() method.
        $this->channel->close();
        $this->connection->close();
    }
}
```

