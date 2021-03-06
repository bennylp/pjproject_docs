
Calls
=====
Calls are represented by Call class.

Subclassing the Call Class
------------------------------------
To use the Call class, normally application SHOULD create its own subclass, such as:

.. code-block:: c++

    class MyCall : public Call
    {
    public:
        MyCall(Account &acc, int call_id = PJSUA_INVALID_ID)
        : Call(acc, call_id)
        { }

        ~MyCall()
        { }

        // Notification when call's state has changed.
        virtual void onCallState(OnCallStateParam &prm);

        // Notification when call's media state has changed.
        virtual void onCallMediaState(OnCallMediaStateParam &prm);
    };

In its subclass, application can implement the call callbacks, which is basically used to process events related to the call, such as call state change or incoming call transfer request.

Making Outgoing Calls
--------------------------------------
Making outgoing call is simple, just invoke makeCall() method of the Call object. Assuming you have the Account object as acc variable and destination URI string in dest_uri, you can initiate outgoing call with the snippet below:

.. code-block:: c++

    Call *call = new MyCall(*acc);
    CallOpParam prm(true); // Use default call settings
    try {
        call->makeCall(dest_uri, prm);
    } catch(Error& err) {
        cout << err.info() << endl;
    }

The snippet above creates a Call object and initiates outgoing call to dest_uri using the default call settings. Subsequent operations to the call can use the method in the call instance, and events to the call will be reported to the callback. More on the callback will be explained a bit later.

Receiving Incoming Calls
--------------------------------------
Incoming calls are reported as onIncomingCall() of the Account class. You must derive a class from the Account class to handle incoming calls.

Below is a sample code of the callback implementation:

.. code-block:: c++

    void MyAccount::onIncomingCall(OnIncomingCallParam &iprm)
    {
        Call *call = new MyCall(*this, iprm.callId);
        CallOpParam prm;
        prm.statusCode = PJSIP_SC_OK;
        call->answer(prm);
    }

For incoming calls, the call instance is created in the callback function as shown above. Application should make sure to store the call instance during the lifetime of the call (that is until the call is disconnected).

Call Properties
----------------
All call properties such as state, media state, remote peer information, etc. are stored as CallInfo class, which can be retrieved from the call object with using getInfo() method of the Call.

Call Disconnection
-------------------
Call disconnection event is a special event since once the callback that reports this event returns, the call is no longer valid and any operations invoked to the call object will raise error exception. Thus, it is recommended to delete the call object inside the callback.

The call disconnection is reported in onCallState() method of Call and it can be detected as follows:

.. code-block:: c++

    void MyCall::onCallState(OnCallStateParam &prm)
    {
        CallInfo ci = getInfo();
        if (ci.state == PJSIP_INV_STATE_DISCONNECTED) {
            /* Delete the call */
            delete this;
        }
    }

Working with Call's Audio Media
-------------------------------------------------
You can only operate with the call's audio media (e.g. connecting the call to the sound device in the conference bridge) when the call's audio media is ready (or active). The changes to the call's media state is reported in onCallMediaState() callback, and if the calls audio media is ready (or active) the function Call.getMedia() will return a valid audio media.

Below is a sample code to connect the call to the sound device when the media is active:

.. code-block:: c++

    void MyCall::onCallMediaState(OnCallMediaStateParam &prm)
    {
        CallInfo ci = getInfo();
        // Iterate all the call medias
        for (unsigned i = 0; i < ci.media.size(); i++) {
            if (ci.media[i].type==PJMEDIA_TYPE_AUDIO && getMedia(i)) {
                AudioMedia *aud_med = (AudioMedia *)getMedia(i);

                // Connect the call audio media to sound device
                AudDevManager& mgr = Endpoint::instance().audDevManager();
                aud_med->startTransmit(mgr.getPlaybackDevMedia());
                mgr.getCaptureDevMedia().startTransmit(*aud_med);
            }
        }
    }

When the audio media becomes inactive (for example when the call is put on hold), there is no need to stop the audio media's transmission to/from the sound device since the call's audio media will be removed automatically from the conference bridge when it's no longer valid, and this will automatically remove all connections to/from the call.

Call Operations
-------------------
You can invoke operations to the Call object, such as hanging up, putting the call on hold, sending re-INVITE, etc. Please see the reference documentation of Call for more info.

Instant Messaging(IM)
---------------------
You can send IM within a call using Call.sendInstantMessage(). The transmission status of outgoing instant messages is reported in Call.onInstantMessageStatus() callback method.

In addition to sending instant messages, you can also send typing indication using Call.sendTypingIndication().

Incoming IM and typing indication received within a call will be reported in the callback functions Call.onInstantMessage() and Call.onTypingIndication().

Alternatively, you can send IM and typing indication outside a call by using Buddy.sendInstantMessage() and Buddy.sendTypingIndication(). For more information, please see Presence documentation.


Class Reference
---------------
Call
++++
.. doxygenclass:: pj::Call
        :project: pjsip
        :members:

Settings
++++++++
.. doxygenstruct:: pj::CallSetting
        :project: pjsip
        :members:
        

Info and Statistics
+++++++++++++++++++
.. doxygenstruct:: pj::CallInfo
        :project: pjsip
        :members:
        
.. doxygenstruct:: pj::CallMediaInfo
        :project: pjsip
        :members:
        
.. doxygenstruct:: pj::StreamInfo
        :project: pjsip
        :members:
        
.. doxygenstruct:: pj::StreamStat
        :project: pjsip
        :members:

.. doxygenstruct:: pj::JbufState
        :project: pjsip
        :members:

.. doxygenstruct:: pj::RtcpStat
        :project: pjsip
        :members:

.. doxygenstruct:: pj::RtcpStreamStat
        :project: pjsip
        :members:

.. doxygenstruct:: pj::MathStat
        :project: pjsip
        :members:

.. doxygenstruct:: pj::MediaTransportInfo
        :project: pjsip
        :members:


Callback Parameters
+++++++++++++++++++
.. doxygenstruct:: pj::OnCallStateParam
        :project: pjsip
        :members:

.. doxygenstruct:: pj::OnCallTsxStateParam
        :project: pjsip
        :members:

.. doxygenstruct:: pj::OnCallMediaStateParam
        :project: pjsip
        :members:

.. doxygenstruct:: pj::OnCallSdpCreatedParam
        :project: pjsip
        :members:

.. doxygenstruct:: pj::OnStreamCreatedParam
        :project: pjsip
        :members:

.. doxygenstruct:: pj::OnStreamDestroyedParam
        :project: pjsip
        :members:

.. doxygenstruct:: pj::OnDtmfDigitParam
        :project: pjsip
        :members:

.. doxygenstruct:: pj::OnCallTransferRequestParam
        :project: pjsip
        :members:

.. doxygenstruct:: pj::OnCallTransferStatusParam
        :project: pjsip
        :members:

.. doxygenstruct:: pj::OnCallReplaceRequestParam
        :project: pjsip
        :members:

.. doxygenstruct:: pj::OnCallReplacedParam
        :project: pjsip
        :members:

.. doxygenstruct:: pj::OnCallRxOfferParam
        :project: pjsip
        :members:

.. doxygenstruct:: pj::OnCallRedirectedParam
        :project: pjsip
        :members:

.. doxygenstruct:: pj::OnCallMediaEventParam
        :project: pjsip
        :members:

.. doxygenstruct:: pj::OnCallMediaTransportStateParam
        :project: pjsip
        :members:

.. doxygenstruct:: pj::OnCreateMediaTransportParam
        :project: pjsip
        :members:

.. doxygenstruct:: pj::CallOpParam
        :project: pjsip
        :members:

.. doxygenstruct:: pj::CallSendRequestParam
        :project: pjsip
        :members:

.. doxygenstruct:: pj::CallVidSetStreamParam
        :project: pjsip
        :members:

Other
+++++
.. doxygenstruct:: pj::MediaEvent
        :project: pjsip
        :members:

.. doxygenstruct:: pj::MediaFmtChangedEvent
        :project: pjsip
        :members:

.. doxygenstruct:: pj::SdpSession
        :project: pjsip
        :members:

.. doxygenstruct:: pj::RtcpSdes
        :project: pjsip


