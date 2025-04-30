# klikkespill
// Klikkespill med animasjon, butikk, grafikk og nivåfordeler
import React, { useState, useEffect, useRef } from 'react';
import { View, Text, Pressable, StyleSheet, Vibration, Image, Alert, ScrollView, Animated } from 'react-native';
import { Audio } from 'expo-av';
import * as Notifications from 'expo-notifications';
import AsyncStorage from '@react-native-async-storage/async-storage';

export default function Klikkespill() {
  const [poeng, setPoeng] = useState(0);
  const [perKlikk, setPerKlikk] = useState(1);
  const [autoKlikk, setAutoKlikk] = useState(0);
  const [afkBonus, setAfkBonus] = useState(1);
  const [nivå, setNivå] = useState(1);
  const [sistAktiv, setSistAktiv] = useState(Date.now());
  const [lyd, setLyd] = useState(null);
  const [bonusSound, setBonusSound] = useState(null);
  const [bakgrunn, setBakgrunn] = useState(require('./bg_default.png'));
  const [prestasjonBonus, setPrestasjonBonus] = useState('');
  const bonusFade = useRef(new Animated.Value(0)).current;

  const scaleAnim = useRef(new Animated.Value(1)).current;
  const blinkAnim = useRef(new Animated.Value(1)).current;

  const butikk = [
    { navn: 'Sterkere klikk (+1)', pris: 10, effekt: () => setPerKlikk(prev => prev + 1), ikon: require('./upgrade_click.png') },
    { navn: 'Auto-klikk (+1/sek)', pris: 50, effekt: () => setAutoKlikk(prev => prev + 1), ikon: require('./upgrade_auto.png') },
    { navn: 'AFK-boost (+1x)', pris: 100, effekt: () => setAfkBonus(prev => prev + 1), ikon: require('./upgrade_afk.png') },
  ];

  useEffect(() => {
    const loadSounds = async () => {
      const clickSound = await Audio.Sound.createAsync(require('./click.mp3'));
      const bonus = await Audio.Sound.createAsync(require('./bonus.mp3'));
      setLyd(clickSound.sound);
      setBonusSound(bonus.sound);
    };

    loadSounds();

    const lastData = async () => {
      const lagretPoeng = await AsyncStorage.getItem('poeng');
      const lagretPerKlikk = await AsyncStorage.getItem('perKlikk');
      const lagretAutoKlikk = await AsyncStorage.getItem('autoKlikk');
      const lagretAfkBonus = await AsyncStorage.getItem('afkBonus');
      const lagretSistAktiv = await AsyncStorage.getItem('sistAktiv');

      if (lagretPoeng !== null) setPoeng(Number(lagretPoeng));
      if (lagretPerKlikk !== null) setPerKlikk(Number(lagretPerKlikk));
      if (lagretAutoKlikk !== null) setAutoKlikk(Number(lagretAutoKlikk));
      if (lagretAfkBonus !== null) setAfkBonus(Number(lagretAfkBonus));
      if (lagretSistAktiv !== null) setSistAktiv(Number(lagretSistAktiv));
    };

    lastData();

    Notifications.setNotificationHandler({
      handleNotification: async () => ({
        shouldShowAlert: true,
        shouldPlaySound: false,
        shouldSetBadge: false,
      }),
    });

    const afkInterval = setInterval(() => {
      const nå = Date.now();
      const minutterAFK = Math.floor((nå - sistAktiv) / (1000 * 60));
      const afkPoeng = minutterAFK * perKlikk * afkBonus;

      if (minutterAFK >= 1) {
        setPoeng(prev => {
          const nyPoeng = prev + afkPoeng;
          AsyncStorage.setItem('poeng', nyPoeng.toString());
          return nyPoeng;
        });
        Notifications.scheduleNotificationAsync({
          content: {
            title: 'AFK Poeng opptjent!',
            body: `Du har fått ${afkPoeng} poeng mens du var borte!`,
          },
          trigger: null,
        });
        setSistAktiv(nå);
        AsyncStorage.setItem('sistAktiv', nå.toString());
      }
    }, 1000 * 60 * 30);

    const autoInterval = setInterval(() => {
      if (autoKlikk > 0) {
        setPoeng(prev => {
          const nyPoeng = prev + autoKlikk;
          AsyncStorage.setItem('poeng', nyPoeng.toString());
          return nyPoeng;
        });
      }
    }, 1000);

    return () => {
      clearInterval(afkInterval);
      clearInterval(autoInterval);
    };
  }, [autoKlikk, afkBonus, perKlikk, sistAktiv]);

  useEffect(() => {
    const nyttNivå = Math.floor(poeng / 100) + 1;

    if (poeng >= 100 && prestasjonBonus !== '🎉 100 poeng!') {
      setPrestasjonBonus('🎉 100 poeng!');
      bonusSound?.replayAsync();
      Animated.sequence([
        Animated.timing(bonusFade, { toValue: 1, duration: 300, useNativeDriver: true }),
        Animated.timing(bonusFade, { toValue: 0, duration: 1000, delay: 1000, useNativeDriver: true }),
      ]).start();
    }
    if (perKlikk >= 5 && prestasjonBonus !== '🎉 5 poeng per klikk!') {
      setPrestasjonBonus('🎉 5 poeng per klikk!');
      bonusSound?.replayAsync();
      Animated.sequence([
        Animated.timing(bonusFade, { toValue: 1, duration: 300, useNativeDriver: true }),
        Animated.timing(bonusFade, { toValue: 0, duration: 1000, delay: 1000, useNativeDriver: true }),
      ]).start();
    }
    if (autoKlikk >= 1 && prestasjonBonus !== '🎉 Auto-klikk låst opp!') {
      setPrestasjonBonus('🎉 Auto-klikk låst opp!');
      bonusSound?.replayAsync();
      Animated.sequence([
        Animated.timing(bonusFade, { toValue: 1, duration: 300, useNativeDriver: true }),
        Animated.timing(bonusFade, { toValue: 0, duration: 1000, delay: 1000, useNativeDriver: true }),
      ]).start();
    }
    if (afkBonus >= 2 && prestasjonBonus !== '🎉 AFK-bonus 2x!') {
      setPrestasjonBonus('🎉 AFK-bonus 2x!');
      bonusSound?.replayAsync();
      Animated.sequence([
        Animated.timing(bonusFade, { toValue: 1, duration: 300, useNativeDriver: true }),
        Animated.timing(bonusFade, { toValue: 0, duration: 1000, delay: 1000, useNativeDriver: true }),
      ]).start();
    }
    if (nyttNivå >= 10 && prestasjonBonus !== '🎉 Nådd nivå 10!') {
      setPrestasjonBonus('🎉 Nådd nivå 10!');
      bonusSound?.replayAsync();
      Animated.sequence([
        Animated.timing(bonusFade, { toValue: 1, duration: 300, useNativeDriver: true }),
        Animated.timing(bonusFade, { toValue: 0, duration: 1000, delay: 1000, useNativeDriver: true }),
      ]).start();
    }

    if (poeng >= 100) setPerKlikk(prev => Math.max(prev, 2));
    if (perKlikk >= 5) setAutoKlikk(prev => Math.max(prev, 1));
    if (autoKlikk >= 1) setAfkBonus(prev => Math.max(prev, 2));

    setNivå(nyttNivå);
    AsyncStorage.setItem('poeng', poeng.toString());

    if (nyttNivå >= 5) setBakgrunn(require('./bg_level5.png'));
    if (nyttNivå >= 10) setBakgrunn(require('./bg_level10.png'));
  }, [poeng]);

  useEffect(() => {
    AsyncStorage.setItem('perKlikk', perKlikk.toString());
    AsyncStorage.setItem('autoKlikk', autoKlikk.toString());
    AsyncStorage.setItem('afkBonus', afkBonus.toString());
  }, [perKlikk, autoKlikk, afkBonus]);

  const klikk = () => {
    lyd?.replayAsync();
    Vibration.vibrate(10);

    Animated.sequence([
      Animated.timing(scaleAnim, { toValue: 0.9, duration: 100, useNativeDriver: true }),
      Animated.timing(scaleAnim, { toValue: 1, duration: 100, useNativeDriver: true }),
    ]).start();

    Animated.sequence([
      Animated.timing(blinkAnim, { toValue: 0, duration: 100, useNativeDriver: true }),
      Animated.timing(blinkAnim, { toValue: 1, duration: 100, useNativeDriver: true }),
    ]).start();

    const nyPoeng = poeng + perKlikk;
    setPoeng(nyPoeng);
    const nå = Date.now();
    setSistAktiv(nå);
    AsyncStorage.setItem('poeng', nyPoeng.toString());
    AsyncStorage.setItem('sistAktiv', nå.toString());
  };

  const kjøpOppgradering = (oppgradering) => {
    if (poeng >= oppgradering.pris) {
      setPoeng(prev => {
        const nyPoeng = prev - oppgradering.pris;
        AsyncStorage.setItem('poeng', nyPoeng.toString());
        return nyPoeng;
      });
      oppgradering.effekt();
    } else {
      Alert.alert('Ikke nok poeng', 'Du har ikke nok poeng til å kjøpe denne oppgraderingen.');
    }
  };

  return (
    <ScrollView contentContainerStyle={styles.container}>
      <Image source={bakgrunn} style={styles.bakgrunn} />
      <Animated.Text style={[styles.poeng, { opacity: blinkAnim }]}>{poeng} poeng</Animated.Text>
      <Text style={styles.nivå}>Nivå {nivå}</Text>
      <Animated.Text style={[styles.bonusTekst, { opacity: bonusFade }]}>{prestasjonBonus}</Animated.Text>
      <Pressable onPress={klikk}>
        <Animated.Image source={require('./donald_figur.png')} style={[styles.figur, { transform: [{ scale: scaleAnim }] }]} />
      </Pressable>

      <Text style={styles.butikkTittel}>Butikk</Text>
      {butikk.map((oppgradering, index) => (
        <Pressable
          key={index}
          style={styles.oppgradering}
          onPress={() => kjøpOppgradering(oppgradering)}
        >
          <Image source={oppgradering.ikon} style={styles.ikon} />
          <Text style={styles.tekst}>{oppgradering.navn} – {oppgradering.pris} poeng</Text>
        </Pressable>
      ))}

      <View style={styles.statsBoks}>
        <Text style={styles.statsTittel}>Statistikk</Text>
        <Text style={styles.statsLinje}>Poeng per klikk: {perKlikk}</Text>
        <Text style={styles.statsLinje}>Auto-klikk/sek: {autoKlikk}</Text>
        <Text style={styles.statsLinje}>AFK-bonus: {afkBonus}x</Text>
        <Text style={styles.statsLinje}>Totalt nivå: {nivå}</Text>
      </View>

      <View style={styles.achievementsBoks}>
        <Text style={styles.statsTittel}>Prestasjoner</Text>
        <Text style={styles.prestasjon}>{poeng >= 100 ? '✅' : '⬜️'} Samle 100 poeng</Text>
        <Text style={styles.prestasjon}>{perKlikk >= 5 ? '✅' : '⬜️'} Oppnå 5 poeng per klikk</Text>
        <Text style={styles.prestasjon}>{autoKlikk >= 1 ? '✅' : '⬜️'} Kjøp auto-klikk</Text>
        <Text style={styles.prestasjon}>{afkBonus >= 2 ? '✅' : '⬜️'} Få AFK-bonus på 2x</Text>
        <Text style={styles.prestasjon}>{nivå >= 10 ? '✅' : '⬜️'} Nå nivå 10</Text>
      </View>
    </ScrollView>
  );
}

const styles = StyleSheet.create({
  bonusTekst: {
    fontSize: 18,
    color: '#FFD700',
    marginBottom: 20,
    fontWeight: 'bold',
  },
  achievementsBoks: {
    backgroundColor: '#222',
    padding: 15,
    borderRadius: 12,
    marginBottom: 30,
    width: '90%',
  },
  prestasjon: {
    color: '#FFFFFF',
    fontSize: 16,
    marginBottom: 4,
  },
  statsBoks: {
    backgroundColor: '#222',
    padding: 15,
    borderRadius: 12,
    marginBottom: 30,
    width: '90%',
  },
  statsTittel: {
    fontSize: 20,
    color: '#FFFFFF',
    marginBottom: 10,
    textAlign: 'center',
  },
  statsLinje: {
    color: '#AAAAAA',
    fontSize: 16,
    marginBottom: 4,
  },
  container: {
    flexGrow: 1,
    backgroundColor: '#121212',
    alignItems: 'center',
    justifyContent: 'center',
    paddingVertical: 40,
  },
  bakgrunn: {
    position: 'absolute',
    width: '100%',
    height: '100%',
    resizeMode: 'cover',
    zIndex: -1,
  },
  poeng: {
    fontSize: 40,
    color: '#FFD700',
    marginBottom: 10,
  },
  nivå: {
    fontSize: 24,
    color: '#00FF00',
    marginBottom: 10,
  },
  figur: {
    width: 200,
    height: 200,
    marginBottom: 30,
  },
  butikkTittel: {
    fontSize: 24,
    color: '#FFFFFF',
    marginBottom: 10,
  },
  oppgradering: {
    backgroundColor: '#1E90FF',
    padding: 15,
    borderRadius: 10,
    marginBottom: 10,
    width: '80%',
    alignItems: 'center',
  },
  ikon: {
    width: 40,
    height: 40,
    marginBottom: 5,
  },
  tekst: {
    color: '#FFFFFF',
    fontSize: 16,
    textAlign: 'center',
  },
});
