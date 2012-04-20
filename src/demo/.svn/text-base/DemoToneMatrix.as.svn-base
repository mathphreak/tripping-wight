package demo
{
	import flash.display.Sprite;
	import flash.events.Event;
	import flash.events.MouseEvent;
	import test.effects.Delay;
	import test.poly.SimplePolySynthVoiceFactory;
	import tonfall.core.Engine;
	import tonfall.core.TimeConversion;
	import tonfall.display.AbstractApplication;
	import tonfall.prefab.poly.PolySynth;

	/**
	 * Simple implementation of the Tonematrix
	 * 
	 * http://lab.andre-michelle.com/tonematrix
	 * 
	 * @author Andre Michelle
	 */
	[SWF(backgroundColor="#000000", frameRate="31", width="768", height="768")]
	public final class DemoToneMatrix extends AbstractApplication
	{
		private const sequencer : TonematrixSequencer = new TonematrixSequencer();
		private const generator : PolySynth = new PolySynth( SimplePolySynthVoiceFactory.INSTANCE );
		private const delay: Delay = new Delay( TimeConversion.barsToMillis(3.0/16.0, Engine.getInstance().bpm) );
		
		private var _container: Sprite;
		private var _selectMode: Boolean;
		
		public function DemoToneMatrix()
		{
			initView();
			initAudio();
		}

		private function initAudio() : void
		{
			engine.processors.push( sequencer );
			engine.processors.push( generator );
			engine.processors.push( delay );

			delay.signalInput = generator.signalOutput;

			sequencer.receiver = generator;

			engine.input = delay.signalOutput;
		}
		
		private function changePattern( u: int, v: int, value: Boolean ): void
		{
			sequencer.pattern[u][v] = value;
		}

		private function initView() : void
		{
			_container = new Sprite();
			_container.x = 128;
			_container.y = 128;
			 addChild( _container );
			
			for( var u: int = 0 ; u < 16 ; ++u )
			{
				for( var v: int = 0 ; v < 16 ; ++v )
				{
					var button: Button = new Button( u, v );
					
					button.x = ( u << 5 );
					button.y = ( v << 5 );
					_container.addChild( button );
				}
			}
			
			addEventListener( MouseEvent.MOUSE_DOWN, mouseDown );
			addEventListener( MouseEvent.MOUSE_MOVE, mouseMove );
		}
		
		override protected function resize( event : Event = null ) : void
		{
			if( null != _container )
			{
				_container.x = ( stage.stageWidth  - 512 ) >> 1;
				_container.y = ( stage.stageHeight - 512 ) >> 1;
			}
		}

		private function mouseMove( event : MouseEvent ) : void
		{
			var button: Button = event.target as Button;
			
			if( event.buttonDown && null != button )
			{
				button.selected = _selectMode;
				
				changePattern( button.u, button.v, button.selected );
			}
		}

		private function mouseDown( event : MouseEvent ) : void
		{
			var button: Button = event.target as Button;
			
			if( null != button )
			{
				button.selected = _selectMode = !button.selected;
				
				changePattern( button.u, button.v, button.selected );
			}
		}
	}
}
import tonfall.core.BlockInfo;
import tonfall.core.Processor;
import tonfall.core.TimeEventNote;

import flash.display.Sprite;

final class Button extends Sprite
{
	private var _u : int;
	private var _v : int;
	
	private var _selected: Boolean;

	public function Button( u : int, v : int )
	{
		_u = u;
		_v = v;
		
		update();
	}

	public function get u() : int
	{
		return _u;
	}

	public function get v() : int
	{
		return _v;
	}

	public function get selected() : Boolean
	{
		return _selected;
	}

	public function set selected( selected : Boolean ) : void
	{
		_selected = selected;
		
		update();
	}
	
	private function update(): void
	{
		graphics.clear();
		graphics.beginFill( _selected ? 0xEDEDED : 0x333333 );
		graphics.drawRoundRect( 2.0, 2.0, 30.0, 30.0, 6, 6 );
		graphics.endFill();
	}
}

final class TonematrixSequencer extends Processor
{
	public static const ToneMatrixNotes : Vector.<int> = Vector.<int>( [96,93,91,89,86,84,81,79,77,74,72,69,67,65,62,60] );
	
	public const pattern: Vector.<Vector.<Boolean>> = new Vector.<Vector.<Boolean>>(16,true);
	
	public var receiver: Processor;
	
	public function TonematrixSequencer()
	{
		for(var i:int = 0 ;i <16;++i)
		{
			pattern[i] = new Vector.<Boolean>(16,true);
		}
	}
	
	override public function process( info : BlockInfo ) : void
	{
		
		var index: int = int( info.barFrom * 16.0 );
		var position: Number = index / 16.0;
		
		while( position < info.barTo )
		{
			if( position >= info.barFrom )
			{
				for( var i: int = 0 ; i < 16 ; ++i )
				{
					if(pattern[index%16][i])
					{
						var event: TimeEventNote = new TimeEventNote();

						event.barPosition = position;
						event.note = ToneMatrixNotes[i];
						event.barDuration = 1.0/16.0;
						receiver.addTimeEvent(event);
					}
				}
			}

			position += 1.0/16.0;
			++index;
		}
	}
}