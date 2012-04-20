package demo
{
	import tonfall.core.IParameterObserver;
	import tonfall.util.Mapping;
	import tonfall.core.Parameter;
	import assets.Loop;
	import assets.Midi;
	import test.poly.SimplePolySynthVoiceFactory;
	import tonfall.core.TimeEventContainerSequencer;
	import tonfall.display.AbstractApplication;
	import tonfall.prefab.audio.ContinuousSyncedLoop;
	import tonfall.prefab.poly.PolySynth;
	import tonfall.prefab.routing.MixingUnit;

	/**
	 * Demoes ContinuousSyncedLoop
	 * 
	 * @author Andre Michelle
	 */
	[SWF(width='512',height='48',backgroundColor='0x1b1b1b',frameRate='32',scriptTimeLimit='255')]
	public final class DemoSyncedLoop extends AbstractApplication
		implements IParameterObserver
	{
		private const TEMPO_MIN: Number = 10.0;
		private const TEMPO_MAX: Number = 300.0;
		
		private const loopPlayer: ContinuousSyncedLoop = new ContinuousSyncedLoop( Loop.INSTANT_COFFEE, Loop.INSTANT_COFFEE.numSamples, 1.0 );
		
		private const sequencer: TimeEventContainerSequencer = new TimeEventContainerSequencer( Midi.BUBBLE_BOBBLE.toTimeEventContainer( 0.0 ) );
		private const generator: PolySynth = new PolySynth( new SimplePolySynthVoiceFactory() );
		
		private const mixer: MixingUnit = new MixingUnit( 2 );
		
		private const parameterTempo: Parameter = new Parameter( 'Tempo', Mapping.mapLinearInv( ( TEMPO_MIN + TEMPO_MAX ) * 0.5, TEMPO_MIN, TEMPO_MAX ) );

		public function DemoSyncedLoop()
		{
			sequencer.timeEventTarget = generator;
			
			mixer.connectAt( generator.signalOutput, 0 );
			mixer.connectAt( loopPlayer.signalOutput, 1 );
			
			engine.bpm = 130.0;
			
			engine.processors.push( sequencer );
			engine.processors.push( generator );
			engine.processors.push( loopPlayer );
			engine.processors.push( mixer );
			
			parameterTempo.addObserver( this );
			addParameterSlider( parameterTempo );
			
			engine.input = mixer.signalOutput;
		}

		public function onParameterChanged( parameter: Parameter ): void
		{
			if( parameter == parameterTempo )
			{
				engine.bpm = Mapping.mapLinear( parameter.value, TEMPO_MIN, TEMPO_MAX );
			}
		}
	}
}