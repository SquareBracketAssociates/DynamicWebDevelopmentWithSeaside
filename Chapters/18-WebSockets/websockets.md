## WebSockets
	visuDivId ifNil: [ visuDivId:= self class nextId ].
	^ visuDivId
]
	visuDivId := anObject
]
	| visuId div callback |
	visuId := self visuDivId.
	div := html div
		id: visuId;
		style: self style;
		attributeAt: #isTelescopeVisu put: 'true';
		attributeAt: #'data-port' put: self webSocketPort;
		class: 'telescopeVisu';
		with: [ html div
				class: 'visualization';
				style: 'height: 100%; width: 100%;'.
			self waitingMessage value: html.
			self renderOptionalButtonsOn: html ];
		yourself.
	callback := WAValueCallback new.
	TLCytoscapeWebSocketDelegate
		registerVisualization: self visualization
		underId: visuId
		withCallBack: callback
		callbackUrl:
			{html actionUrl asString.
			(div storeCallback: callback)}
]
	html div
		class: 'fitButton';
		with: [ self renderResetButtonOn: html.
			self exportStrategy renderDownloadButtonForVisu: self visuDivId on: html ]
]
	html anchor
		onClick: 'telescope.visuWithId(' , self visuDivId asString , ').fit();';
		with: 'Reset'
]
	instVars: 'visualizationByIdDictionary websocketByVisu'
	classInstVars: 'singleton development serverPort clientPort'
	self ensureServerIsRunning.
	self singleton delegate
		registerVisualization: aTLVisualization
		underId: aDivId
		withCallBack: aCallBack
		callbackUrl: callbackUrl
]
   super initialize.
   self visualizationByIdDictionary: Dictionary new.
   self websocketByVisu: Dictionary new.
   self
      handler: [ :webSocket | 
   		[ 
   			[ webSocket runWith: [ :message | self onMessageReceived: message webSocket: webSocket ] ]
   				on: ConnectionClosed
   				do: [ self freeResourcesFor: webSocket ] ]
   					on: PrimitiveFailed
   					do: [ self class restart ] ]
   ]