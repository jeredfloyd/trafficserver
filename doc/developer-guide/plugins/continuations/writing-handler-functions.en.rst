.. Licensed to the Apache Software Foundation (ASF) under one
   or more contributor license agreements.  See the NOTICE file
   distributed with this work for additional information
   regarding copyright ownership.  The ASF licenses this file
   to you under the Apache License, Version 2.0 (the
   "License"); you may not use this file except in compliance
   with the License.  You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing,
   software distributed under the License is distributed on an
   "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
   KIND, either express or implied.  See the License for the
   specific language governing permissions and limitations
   under the License.

.. include:: ../../../common.defs

.. _developer-plugins-continuations-handler-functions:

Writing Handler Functions
*************************

.. default-domain:: c

The handler function is the key component of a continuation. It is
supposed to examine the event and event data, and then do something
appropriate. The probable action might be to schedule another event for
the continuation to received, to open up a connection to a server, or
simply to destroy itself.

The continuation's handler function is a function of type
:type:`TSEventFunc`. Its arguments are a continuation, an event, and a
pointer to some data (this data is passed to the continuation by the
caller - do not confuse this data with the continuation's own data,
associated by :func:`TSContDataSet`). When the continuation is called back,
the continuation and an event are passed to the handler function. The
continuation is a handle to the same continuation that is invoked. The
handler function typically has a switch statement to handle the events
it receives:

.. code-block:: c

   static int some_handler (TScont contp, TSEvent event, void *edata)
   {
      // .....
      switch(event) {
         case TS_EVENT_SOME_EVENT_1:
            do_some_thing_1;
            return;
         case TS_EVENT_SOME_EVENT_2:
            do_some_thing_2;
            return;
         case TS_EVENT_SOME_EVENT_3:
            do_some_thing_3;
            return;
         default: break;
      }
      return 0;
   }

.. caution::

   You might notice that a continuation cannot determine if more events are
   "in flight" toward it. Do not use :func:`TSContDestroy` to delete a
   continuation before you make sure that all incoming events, such as
   those sent because of :func:`TSHttpTxnHookAdd`, have been handled.

.. caution::

   TS_HTTP_SEND_REQUEST_HDR_HOOK may callback several times when the
   OS crashed. Be careful to use functions such as TSContDestroy in
   TS_HTTP_SEND_REQUEST_HDR_HOOK hook.

The following table lists events and the corresponding type of
`void* data` passed to handler functions:

============================================= ============================================ ==========================
Event                                         Event Sender                                 Data Type
============================================= ============================================ ==========================
:macro:`TS_EVENT_HTTP_READ_REQUEST_HDR`       :macro:`TS_HTTP_READ_REQUEST_HDR_HOOK`       :type:`TSHttpTxn`
:macro:`TS_EVENT_HTTP_PRE_REMAP`              :macro:`TS_HTTP_PRE_REMAP_HOOK`              :type:`TSHttpTxn`
:macro:`TS_EVENT_HTTP_OS_DNS`                 :macro:`TS_HTTP_OS_DNS_HOOK`                 :type:`TSHttpTxn`
:macro:`TS_EVENT_HTTP_SEND_REQUEST_HDR`       :macro:`TS_HTTP_SEND_REQUEST_HDR_HOOK`       :type:`TSHttpTxn`
:macro:`TS_EVENT_HTTP_READ_CACHE_HDR`         :macro:`TS_HTTP_READ_CACHE_HDR_HOOK`         :type:`TSHttpTxn`
:macro:`TS_EVENT_HTTP_READ_RESPONSE_HDR`      :macro:`TS_HTTP_READ_RESPONSE_HDR_HOOK`      :type:`TSHttpTxn`
:macro:`TS_EVENT_HTTP_SEND_RESPONSE_HDR`      :macro:`TS_HTTP_SEND_RESPONSE_HDR_HOOK`      :type:`TSHttpTxn`
:macro:`TS_EVENT_HTTP_SELECT_ALT`             :macro:`TS_HTTP_SELECT_ALT_HOOK`             :type:`TSHttpTxn`
:macro:`TS_EVENT_HTTP_TXN_START`              :macro:`TS_HTTP_TXN_START_HOOK`              :type:`TSHttpTxn`
:macro:`TS_EVENT_HTTP_TXN_CLOSE`              :macro:`TS_HTTP_TXN_CLOSE_HOOK`              :type:`TSHttpTxn`
:macro:`TS_EVENT_HTTP_SSN_START`              :macro:`TS_HTTP_SSN_START_HOOK`              :type:`TSHttpSsn`
:macro:`TS_EVENT_HTTP_SSN_CLOSE`              :macro:`TS_HTTP_SSN_CLOSE_HOOK`              :type:`TSHttpSsn`
:macro:`TS_EVENT_NONE`
:macro:`TS_EVENT_CACHE_LOOKUP_COMPLETE`       :macro:`TS_HTTP_CACHE_LOOKUP_COMPLETE_HOOK`  :type:`TSHttpTxn`
:macro:`TS_EVENT_IMMEDIATE`                   :func:`TSVConnClose`
                                              :func:`TSVIOReenable`
                                              :func:`TSContSchedule`
                                              :func:`TSContScheduleOnPool`
                                              :func:`TSContScheduleOnThread`
:macro:`TS_EVENT_IMMEDIATE`                   :macro:`TS_HTTP_REQUEST_TRANSFORM_HOOK`
:macro:`TS_EVENT_IMMEDIATE`                   :macro:`TS_HTTP_RESPONSE_TRANSFORM_HOOK`
:macro:`TS_EVENT_CACHE_OPEN_READ`             :func:`TSCacheRead`                          Cache VC
:macro:`TS_EVENT_CACHE_OPEN_READ_FAILED`      :func:`TSCacheRead`                          TS_CACHE_ERROR code
:macro:`TS_EVENT_CACHE_OPEN_WRITE`            :func:`TSCacheWrite`                         Cache VC
:macro:`TS_EVENT_CACHE_OPEN_WRITE_FAILED`     :func:`TSCacheWrite`                         TS_CACHE_ERROR code
:macro:`TS_EVENT_CACHE_REMOVE`                :func:`TSCacheRemove`
:macro:`TS_EVENT_CACHE_REMOVE_FAILED`         :func:`TSCacheRemove`                        TS_CACHE_ERROR code
:macro:`TS_EVENT_NET_ACCEPT`                  :func:`TSNetAccept`                          :type:`TSNetVConnection`
                                              :func:`TSHttpTxnServerIntercept`
                                              :func:`TSHttpTxnIntercept`
:macro:`TS_EVENT_NET_ACCEPT_FAILED`           :func:`TSNetAccept`
                                              :func:`TSHttpTxnServerIntercept`
                                              :func:`TSHttpTxnIntercept`
:macro:`TS_EVENT_HOST_LOOKUP`                 :func:`TSHostLookup`                         :type:`TSHostLookupResult`
:macro:`TS_EVENT_TIMEOUT`                     :func:`TSContSchedule`
                                              :func:`TSContScheduleOnPool`
                                              :func:`TSContScheduleOnThread`
:macro:`TS_EVENT_ERROR`
:macro:`TS_EVENT_VCONN_READ_READY`            :func:`TSVConnRead`                          :type:`TSVIO`
:macro:`TS_EVENT_VCONN_WRITE_READY`           :func:`TSVConnWrite`                         :type:`TSVIO`
:macro:`TS_EVENT_VCONN_READ_COMPLETE`         :func:`TSVConnRead`                          :type:`TSVIO`
:macro:`TS_EVENT_VCONN_WRITE_COMPLETE`        :func:`TSVConnWrite`                         :type:`TSVIO`
:macro:`TS_EVENT_VCONN_EOS`                   :func:`TSVConnRead`                          :type:`TSVIO`
:macro:`TS_EVENT_NET_CONNECT`                 :func:`TSNetConnect`                         :type:`TSVConn`
:macro:`TS_EVENT_NET_CONNECT_FAILED`          :func:`TSNetConnect`                         :type:`TSVConn`
:macro:`TS_EVENT_HTTP_CONTINUE`
:macro:`TS_EVENT_HTTP_ERROR`
:macro:`TS_EVENT_MGMT_UPDATE`                 :func:`TSMgmtUpdateRegister`
============================================= ============================================ ==========================

The continuation functions are listed below:

-  :func:`TSContCall`
-  :func:`TSContCreate`
-  :func:`TSContDataGet`
-  :func:`TSContDataSet`
-  :func:`TSContDestroy`
-  :func:`TSContMutexGet`
-  :func:`TSContSchedule`
-  :func:`TSContScheduleOnPool`
-  :func:`TSContScheduleOnThread`

When a handler function blocks, it blocks the event thread running it.  This blocks all the continuations (internal ones
along with those of plugins) in the event thread's queue.  This may increase the worst-case latency for HTTP request
processing.  If there is enough blocking, this could increase CPU idle time, which may reduce proxy throughput.  The
Au test **polite_hook_wait** illustrates a method for using dynamic threading to do a blocking call without blocking
any handler function.  But the overhead of this method may cancel out the performance improvement, if blocking times
are short.
