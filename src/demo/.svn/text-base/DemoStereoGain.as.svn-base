package demo
{
	import tonfall.display.AbstractApplication;

	import flash.events.Event;

	/**
	 * Demoes StereoMatrix (Does not interpolate)
	 * 
	 * @author Andre Michelle
	 */
	[SWF(backgroundColor="#FFFFFF", frameRate="31", width="800", height="128")]
	public final class DemoStereoGain extends AbstractApplication
	{
		private const processor: StereoSine = new StereoSine();
		
		public function DemoStereoGain()
		{
			stage.addEventListener( Event.ENTER_FRAME, enterFrame );
			
			engine.processors.push( processor );

			engine.input = processor.output;
		}
		
		override protected function resize( event: Event = null ): void
		{
			const wHalf: Number = stage.stageWidth * 0.5;
			const height: Number = stage.stageHeight;

			graphics.clear();
			graphics.lineStyle( 0.0, 0xFF0000 );
			graphics.moveTo( wHalf, 0.0 );
			graphics.lineTo( wHalf, height );
			
			super.resize( event );
		}

		private function enterFrame( event: Event ): void
		{
			const wHalf: Number = stage.stageWidth * 0.5;

			const pan: Number = ( mouseX - wHalf ) / wHalf;
			
			processor.matrix.linear( 0.5, pan );
		}
	}
}
import tonfall.core.Signal;
import tonfall.core.SignalBuffer;
import tonfall.core.SignalProcessor;
import tonfall.util.StereoMatrix;

final class StereoSine extends SignalProcessor
{
	public const output: SignalBuffer = new SignalBuffer();
	
	public const matrix: StereoMatrix = new StereoMatrix();
	
	private var _phaseL: Number;
	private var _phaseR: Number;
	
	private var _phaseIncrL: Number;
	private var _phaseIncrR: Number;

	public function StereoSine()
	{
		_phaseL = 0.0;
		_phaseR = 0.0;
		
		_phaseIncrL = 440.0 / 44100.0;
		_phaseIncrR = 330.0 / 44100.0;
	}
	
	override protected function processSignals( numSignals: int ): void
	{
		var s: Signal = output.current;
		
		for( var i: int = 0 ; i < numSignals ; ++i )
		{
			s.l = Math.sin( _phaseL * Math.PI * 2.0 );
			s.r = Math.sin( _phaseR * Math.PI * 2.0 );
			
			_phaseL += _phaseIncrL;
			_phaseL -= Math.floor( _phaseL );
			_phaseR += _phaseIncrR;
			_phaseR -= Math.floor( _phaseR );
			
			matrix.transform( s, s );
			
			s = s.next;
		}
	}
}